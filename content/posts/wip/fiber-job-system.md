---
title: "Implementing a Fiber-Based Job System"
date: 2026-01-24
draft: true
---

This implementation is almost entirely based on [parallelizing the Naughty Dog engine using fibers](https://www.youtube.com/watch?v=Kvsvd67XUKw) by Christian Gyrling with inspiration from [parallelizing the physics solver in Teardown](https://www.youtube.com/watch?v=Kvsvd67XUKw) by Dennis Gustaffson. This is just a post about my approach because I figure it's quite interesting.

# Introduction

Feel free to skip this part if you're already familiar with what a job system is or why you'd want one.

Alright so a common problem in software engineering, but especially real-time performance-heavy applications like games and renderers, is question of how to parallelize the code in an effective and scalable fashion.

Essentially, how can we make the most effective use of all threads on the CPU with minimal wasted time?

... (photo of thread usage graph but not being used effectively)

Let's assume you're parallelizing your game. Starting from the beginning, the first solution you might come up with is to run your simulation and renderer on seperate threads which get synced up on a "master" controller thread running the main game loop.

```cpp
start_thread(simulation);
start_thread(rendering);

while (running) {
	signal_thread(simulation);
	barrier(simulation, rendering);
	upload_render_data();
	signal_thread(rendering);
}

join_all_threads();
```

This is obviously very simplified but you get the idea.

This is very far from ideal. It's clearly very inextensible, and rigid. We can't easily add another system without opening another thread and even if we did it essentially means we only get to have like 10 systems before we run out of threads. Imagine if we want to use something as simple as a parallel for?

Additionally, While it looks parallel it's still incredibly linear as all the internals of the simulation and rendering code will run serially.

Finally, we have terrible CPU utilization as the main thread has to wait (`barrier(simulation, rendering)`) until both threads finish, which means if either one of them finishes before the other precious CPU cycles get wasted. Threads being underutilized is the opposite of what we want when we start multi-threading.

So no dice.

## Jobs

Lets start from the beginning again. We might try to reframe how we think about each system. The simulation and rendering systems are really just some amount of work to be completed by some point. We might call them "jobs".

At its core, a job is just a function call + some parameters:

```cpp
typedef EntryPoint(uptr param);

struct JobDecl {
	EntryPoint *entry_point;
	uptr param;
};
```

We also need some way to "wait" on a job to finish so lets introduce a job handle of some kind so we can keep track of them.

We want as simple an API as possible to make parallelizing our program as easy and seamless as possible:

```cpp
JobHandle kick_job(const JobDecl &job);
void wait_on_job(const JobHandle &handle);
```

In this case, "kicking" a job just means "fire and forget". We give it the `JobDecl` and then we just assume it'll get ran at somepoint, somehow, and we can wait on it when we  need to.

However, it's still pretty bad.

## Job Counters

The problem is that job handles kinda suck. We could potentially be dealing with hundreds if not thousands of jobs per-frame and keeping track of all those handles sounds painful. Imagine having to maintain a list of 1000 job handles just to wait on a batch of jobs that you want to kick off, when you would have waited on them all at the same time anyway.

Enter, job counters!

A job counter is an incredibly simple idea. It's an atomic object that can be incremented or decremented by some amount. We can then wait on a counter until it hits a certain value.

So what we can do is that when a job is kicked off we increment the counter associated with that job, and when the job is complete we decrement the counter. This way we just wait until it hits zero again after kicking off a job to synchronize.

What makes job counters great is that a single counter can be assigned to any number of jobs, and then waiting on that counter will wait until all associated jobs are finished.

Changing our API to reflect this:

```cpp
JobCounter *alloc_counter();
void free_counter(JobCounter **counter);

void wait_on_counter(const JobCounter *counter, u32 value = 0);
void wait_on_counter_and_free(const JobCounter *counter, u32 value = 0);

void kick_job(const JobDecl &decl, JobCounter **counter);
void kick_job_batch(const JobDecl *decls, u32 count, JobCounter **counter);
```

## Fibers

Our job system still has a glaring issue - jobs waiting on jobs. In order to fully parallelize our program we need jobs to be able to start jobs themselves, obviously.

However, when waiting on their jobs to complete the thread executing said job completely stalls until it's finished, making a job call no more performant than just running the code in the original job in the first place.

Imagine this scenario:

```cpp
void foo(uptr param)
{
	// ...

	JobDecl bar_job = {};
	bar_job.entry_point = bar;

	JobCounter *c;
	kick_job(bar_job, &c);
	wait_on_counter_and_free(c);

	// ...
}
```

In this case, the thread running `foo` will just stall on `wait_on_counter_and_free` until `bar` finishes on another thread. This is *terrible* because we're occupying two threads when we could just be doing:

```cpp
void foo(uptr param)
{
	// ...

	bar();

	// ...
}

```

In fact, it's even worse than that because imagine hypothetically there was only one worker thread, the one running `foo`.

Well, `foo` would add `bar` to the job queue but `bar` would never get executed because the thread would be stuck executing `foo` which will never complete and thus `bar` never gets fetched from the job queue, so we'd enter a deadlock!

What we need is some way for the worker thread to switch to working on another job when it detects it's waiting on a counter, without losing the stack context so it can switch back to that position after the job is finished and continue executing the original job.

The solution to this proposed by Naughty Dog is using fibers. Fibers are special because every fiber maintains it's own stack.

To reflect this change, I rename the `wait_on_counter` functions to `yield_on_counter`.

# API

To start, I laid out the basic API that I wanted. The job system has to be frictionless to work with and allow for jobs to be launched from pretty much anywhere so:

```cpp
typedef JOB_ENTRY_POINT(EntryPoint);
typedef JOB_PARALLEL_FOR(EntryPointParallelFor);

enum JobPriority {
	PRIORITY_LOW,
	PRIORITY_NORMAL,
	PRIORITY_HIGH,
	PRIORITY_MAX_ENUM
};

struct JobDecl {
	EntryPoint *entry_point;
	uptr param;
	JobPriority priority;
};

// Atomic lockless job counter.
struct JobCounter;

// Functions for starting and shutting down the job system.
void init(u32 initial_worker_count);
void shutdown();

// Creating and destroying counters.
JobCounter *alloc_counter(u32 initial_count = 0);
void free_counter(JobCounter *counter);

// Call to yield here until counter reaches desired value.
void yield_on_counter(JobCounter *counter, u32 value = 0);
void yield_on_counter_and_free(JobCounter *counter, u32 value = 0);

// Core functions for starting jobs.
void kick_job(const JobDecl *job, JobCounter **counter);
void kick_job_batch(const JobDecl **jobs, u32 count, JobCounter **counter);

// Split a for loop across multiple jobs in parallel.
void parallel_for(EntryPointParallelFor *fn, u32 count, JobPriority priority = PRIORITY_NORMAL, u64 batch_size = 64);

// Switch to using a lockless spin instead of a mutex when waiting on jobs.
bool is_spin_mode_enabled();
void set_spin_mode(bool enabled);

// Utils.
u32 get_current_worker_id();
bool is_main_thread();
```

I've also got some macros for utility:

```cpp
#if defined(__x86_64__)
# define JOB_SPIN_PAUSE() _mm_pause()
#else
# define JOB_SPIN_PAUSE() std::this_thread::yield()
#endif

#define JOB_ENTRY_POINT(fname_) void fname_(uptr param)
#define JOB_PARALLEL_FOR(fname_, index_) void fname_(u64 index_)
```

```cpp
JOB_ENTRY_POINT(foo)
{
	MyJobParams *p = (MyJobParams *)param;
	log::info("(%d) %s", job::get_current_worker_id(), p->message_or_whatever);
}

JOB_PARALLEL_FOR(bar, i)
{
	log::info("(%d) %d", job::get_current_worker_id(), i);
}
```

# Implementation

To execute these jobs, we make it so all threads apart from the master controller thread are added to a "thread pool". Each one of these new "workers" will check if a new job is available. If it isn't it will wait until one is, and if it is then it will pull it from the job queue and execute it by calling the entry point with the params.

...

# Potential Improvements

...

# Closing Thoughts

Anyway, I'm pretty happy with the end result, and I hope you found this helpful in any way. Thank you for reading :).
