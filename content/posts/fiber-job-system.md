---
title: "Implementing a Fiber-Based Job System"
date: 2026-08-18
categories:
  - Programming
---

TODO: THIS POST IS UNDER CONSTRUCTION. AREAS ARE MARKED 'TODO' IF BEING WORKED ON. REGARDLESS I'M PUBLISHING IT ANYWAY.

This implementation is based on [parallelizing the Naughty Dog engine using fibers](https://www.youtube.com/watch?v=Kvsvd67XUKw) by Christian Gyrling with inspiration from [parallelizing the physics solver in Teardown](https://www.youtube.com/watch?v=Kvsvd67XUKw) by Dennis Gustaffson.

This is just a post about my approach because I think it's quite interesting.


# Introduction

Feel free to skip this part if you're already familiar with what a job system is or why you'd want one.

Alright so a common problem in software engineering, but especially real-time performance-heavy applications like games and renderers, is question of how to parallelize the code in an effective and scalable fashion.

Essentially, how can we make the most effective use of all threads on the CPU with minimal wasted time?

TODO (photo of thread usage graph but not being used effectively)

Let's assume you're parallelizing your game. Starting from the beginning, the first solution you might come up with is to run your simulation and renderer on seperate threads which get synced up on a "master" controller thread running the main game loop.

```c
// start our logic and render threads
StartThread(simulation);
StartThread(rendering);

while (running)
{
	// notify logic thread to start next iteration
	SignalThread(simulation);

	// wait until both logic and rendering are free
	Barrier(simulation, rendering);

	// upload the data to render
	UploadSimulationDataToRender();

	// tell rendering to start the next iteration
	SignalThread(rendering);
}

// join both threads
JoinThread(simulation);
JoinThread(rendering);
```

This is obviously very simplified but you get the idea.

This is very far from ideal. It's clearly very inextensible and rigid. We can't easily add another system without opening another thread and even if we did it essentially means we only get to have like 10-ish systems before we run out of threads, and ideally we wanna have each thread running on a seperate core. Imagine if we want to use something as simple as a parallel for?

Additionally, while it looks parallel it's still incredibly linear as all the internals of the simulation and rendering code will run serially anyway. Lots of the simulation, and rendering phases could both be parallelized.

Finally, we have terrible CPU utilization as the main thread has to wait (`barrier(simulation, rendering)`) until both threads finish, which means if either one of them finishes before the other precious CPU cycles get wasted. Threads being underutilized is the opposite of what we want when we start multi-threading.

So no dice.

## Jobs

Lets start from the beginning again. We might try to reframe how we think about each system. The simulation and rendering systems are really just some amount of work to be completed by some point. We might call them "jobs".

At its core, a job is just a function call + some parameters:

```cpp
typedef EntryPointFn(void *param);

typedef struct JobDecl JobDecl;
struct JobDecl
{
	EntryPointFn *entry_point;
	void *param;
};
```

We also need some way to "wait" on a job to finish so lets introduce a job handle of some kind so we can keep track of them.

We want as simple an API as possible to make parallelizing our program as easy and seamless as possible:

```c
JobHandle KickJob(const JobDecl *job);
void WaitForJob(JobHandle handle);
```

In this case, "kicking" a job just means "fire and forget". We give it the `JobDecl` and then we just assume it'll get ran at some point, somehow, and we can wait on it when we need to.

This is far better than our previous solution, but it's still pretty bad.

The problem is that job handles kinda suck. We could potentially be dealing with hundreds if not thousands of jobs per-frame and keeping track of all those handles sounds painful. Imagine having to maintain a list of 1000 job handles just to wait on a batch of jobs that you want to kick off, when you would have waited on them all at the same time anyway. We don't need that level of granularity and the bureocracy that goes along with it.

Turns out we can fix this!


## Job Counters

Enter, job counters!

A job counter is an incredibly simple idea. It's an atomic counter that can be incremented or decremented by some amount. We can then wait on a counter until it hits a certain value.

So what we can do is that when a job is kicked off we increment the counter associated with that job, and when the job is complete we decrement the counter. This way we just wait until it hits zero again after kicking off a job to synchronize.

What makes job counters great is that a single counter can be assigned to any number of jobs, and then waiting on that counter will wait until all associated jobs are finished.

Changing our API to reflect this:

```cpp
JobCounter *AllocCounter();
void FreeCounter(JobCounter *counter);

void WaitOnCounter(const JobCounter *counter, u32 value);
void WaitOnCounterAndFree(const JobCounter *counter, u32 value);

void KickJob(const JobDecl *decl, JobCounter *counter);
void KickBatch(const JobDecl *decls, u32 count, JobCounter *counter);
```

So now our API looks a little like this:

```c
JobDecl decl = { ... };

JobCounter *counter = AllocCounter();

// kicking n jobs automatically adds n to the counter
KickBatch(&decl, 100, counter);

// wait on all 100 jobs!
WaitOnCounterAndFree(counter, 0);
```


## Fibers

Our job system still has a glaring issue - jobs waiting on jobs. In order to fully parallelize our program we need jobs to be able to start jobs themselves, obviously.

However, when waiting on their jobs to complete the thread executing said job completely stalls until it's finished, making a job call no more performant than just running the code in the original job in the first place.

Imagine this scenario:

```c
void BarJob(void *param);

void FooJob(void *param)
{
	// ...

	JobDecl job = { BarJob, NULL };

	JobCounter *c = AllocCounter();
	KickJob(&job, c);
	WaitOnCounterAndFree(c);

	// ...
}
```

In this case, the thread running `FooJob` will just stall on `WaitOnCounterAndFree` until `BarJob` finishes on another thread. This is *terrible* because we're occupying two threads when we could just be doing:

```c
void FooJob(uptr param)
{
	// ...

	BarJob();

	// ...
}

```

In fact, it's even *worse* than that because imagine hypothetically there was only one worker thread left, the one running `FooJob`.

Well, `FooJob` would add `BarJob` to the job queue but `BarJob` would never get executed because the thread would be stuck executing `FooJob` which will never complete and thus `BarJob` never gets fetched from the job queue, so we'd enter a deadlock!

What we need is some way for the worker thread to switch to working on another job when it detects it's waiting on a counter, without losing the stack context so it can switch back to that position after the job is finished and continue executing the original job. I.e: We need context switching!

The solution to this proposed by Naughty Dog is using fibers. Fibers are special because every fiber maintains it's own stack.

To reflect this change, I rename the `WaitOnCounter` functions to `YieldOnCounter`.


# API

To start, I lay out the basic API that we want. The job system has to be frictionless to work with and allow for jobs to be launched from pretty much anywhere:

TODO: RIGHT NOW I JUST COPY PASTED FROM MY ENGINE, TIDY AND CLEAN THIS UP TO BE MORE GENERIC AND CLEARER TO THE READER

```c

#ifdef __x86_64__
# include <immintrin.h>
# define OS_SPIN_PAUSE() _mm_pause()
#else
# define OS_SPIN_PAUSE() do { } while (0)
#endif

#define J_ENTRY_POINT_DEF(fn) void fn(void *param)
#define J_PARALLEL_FOR_DEF(fn) void fn(u32 index)

typedef J_ENTRY_POINT_DEF(J_EntryPointFn);
typedef J_PARALLEL_FOR_DEF(J_EntryForFn);

typedef enum J_Priority
{
	J_Priority_Low,
	J_Priority_Normal,
	J_Priority_High,
	J_Priority_COUNT
}
J_Priority;

typedef u32 J_Flags;
enum
{
	J_Flag_None           = 0,
	J_Flag_MainThreadOnly = 1 << 0
};

typedef struct J_Decl J_Decl;
struct J_Decl
{
	J_EntryPointFn *EntryPoint;
	void *param;
	J_Priority priority;
	J_Flags flags;
};

#define J_W32_MAX_JOBS_PER_QUEUE         512
#define J_W32_MAX_CONCURRENT_FIBERS      128
#define J_W32_MAX_WORKERS                32
#define J_W32_COUNTER_MAX_WAITING        128
#define J_W32_FIBER_SCRATCH_SIZE         Megabytes(8)
#define J_W32_FIBER_SCRATCH_RING_SIZE    2

typedef struct J_W32_Context J_W32_Context;
struct J_W32_Context
{
	u32 worker_id;
	i32 fiber_id;
};

typedef struct J_W32_Fiber J_W32_Fiber;

typedef struct J_W32_Counter J_W32_Counter;
struct J_W32_Counter
{
	i32 atomic_count;
	i32 atomic_spinlock;

	// Fibers that are blocked waiting on this counter.
	u32 waiting_count;
	J_W32_Fiber *waiting[J_W32_COUNTER_MAX_WAITING];
};

typedef struct J_W32_Fiber J_W32_Fiber;
struct J_W32_Fiber
{
	u32 id;
	J_W32_Fiber *next_free;
	OS_Handle handle;
	J_EntryPointFn *EntryPoint;
	void *param;
	J_Priority priority;
	J_W32_Counter *counter;
	b32 finished;
	Arena scratch_arenas[J_W32_FIBER_SCRATCH_RING_SIZE];
};

typedef struct J_W32_FiberMutex J_W32_FiberMutex;
struct J_W32_FiberMutex
{
	i32 atomic_mutex_state;
	i32 atomic_spinlock;
	u32 waiting_count;
	J_W32_Fiber *waiting[J_W32_COUNTER_MAX_WAITING];
};

typedef struct J_W32_FiberCondVar J_W32_FiberCondVar;
struct J_W32_FiberCondVar
{
	i32 atomic_spinlock;
	u32 waiting_count;
	J_W32_Fiber *waiting[J_W32_COUNTER_MAX_WAITING];
};

typedef struct J_W32_Request J_W32_Request;
struct J_W32_Request
{
	J_EntryPointFn *EntryPoint;
	void *param;
	J_Priority priority;
	J_Flags flags;
	J_W32_Counter *counter;
};

typedef struct J_W32_Queue J_W32_Queue;
struct J_W32_Queue
{
	i32 atomic_spinlock;
	
	J_W32_Request requests[J_W32_MAX_JOBS_PER_QUEUE];
	J_W32_Fiber *waiting[J_W32_MAX_JOBS_PER_QUEUE];

	i32 atomic_taken_task_count;
	i32 atomic_added_task_count;
	
	i32 atomic_taken_waiting_count;
	i32 atomic_added_waiting_count;
};

typedef struct J_W32_Scheduler J_W32_Scheduler;

typedef struct J_W32_Worker J_W32_Worker;
struct J_W32_Worker
{
	u32 id;
	
	OS_Handle thread_handle;
	OS_Handle fiber_handle; // The scheduler fiber for this current worker thread.

	J_W32_Fiber *current_fiber; // The fiber currently executing on this worker.

	// you know what fuck you.
	J_W32_Scheduler *scheduler;
};

typedef struct J_W32_Scheduler J_W32_Scheduler;
struct J_W32_Scheduler
{
	LOG_Channel log_channel;
	
	i32 atomic_running;
	i32 atomic_spin_mode;

	OS_Handle thread_mutex;
	OS_Handle thread_cond_begin;

	J_W32_Queue main_thread_queue;
	J_W32_Queue queues[J_Priority_COUNT];
	
	Arena fallback_scratch_ring[J_W32_FIBER_SCRATCH_RING_SIZE];
	
	u32 worker_count;
	J_W32_Worker workers[J_W32_MAX_WORKERS];

	J_W32_Fiber atomic_fiber_storage[J_W32_MAX_CONCURRENT_FIBERS];

	i32 fiber_pool_spinlock;
	J_W32_Fiber *fiber_pool_head;
	
	void (*OnMainThreadIdle)(void *ctx);
	void *main_thread_idle_ctx;

	//u32 tls_worker_slot;
};


/* ==================================================
   HELPERS
   ================================================== */

internal void J_W32_SpinModeEnable(J_W32_Scheduler *scheduler);
internal void J_W32_SpinModeDisable(J_W32_Scheduler *scheduler);

internal b32 J_W32_IsMainThread(J_W32_Scheduler *scheduler);

internal void J_W32_FiberYield(J_W32_Scheduler *scheduler);
internal void J_W32_FiberCompleted(J_W32_Scheduler *scheduler);
internal J_W32_Fiber *J_W32_FiberFetchFree(J_W32_Scheduler *scheduler);
internal void J_W32_FiberReturn(J_W32_Scheduler *scheduler, J_W32_Fiber *fiber);
internal OS_Handle J_W32_GetCurrentFiberHandle(J_W32_Scheduler *scheduler);

internal J_W32_Request *J_W32_TryGetRequest(J_W32_Scheduler *scheduler);
internal J_W32_Fiber *J_W32_TryGetWaitingFiber(J_W32_Scheduler *scheduler);

internal b32 J_W32_RequestAvailable(J_W32_Scheduler *scheduler);


/* ==================================================
   CORE
   ================================================== */

internal void J_W32_Init(J_W32_Scheduler *scheduler, LOG_Channel log_channel);
internal void J_W32_Shutdown(J_W32_Scheduler *scheduler);

internal void J_W32_SchedulerThreadEntry(void *param);
internal void J_W32_FiberEntry(void *param);

internal void J_W32_Enter(J_W32_Scheduler *scheduler, void (*OnMainThreadIdle)(void *ctx), void *main_thread_idle_ctx);
internal void J_W32_Halt(J_W32_Scheduler *scheduler);

internal J_W32_Context J_W32_GetContext(J_W32_Scheduler *scheduler);

/* ==================================================
   SYNCHRONISATION
   ================================================== */

internal void J_W32_CounterInit(J_W32_Counter *counter, i32 initial_count);
internal void J_W32_CounterIncrement(J_W32_Counter *counter, i32 n);
internal void J_W32_CounterDecrement(J_W32_Scheduler *scheduler, J_W32_Counter *counter, i32 n);
internal u32  J_W32_CounterValue(J_W32_Counter *counter);

internal void J_W32_FiberMutexLock(J_W32_Scheduler *scheduler, J_W32_FiberMutex *m);
internal void J_W32_FiberMutexUnlock(J_W32_Scheduler *scheduler, J_W32_FiberMutex *m);

internal void J_W32_FiberCondVarWait(J_W32_Scheduler *scheduler, J_W32_FiberCondVar *cv, J_W32_FiberMutex *mutex);
internal void J_W32_FiberCondVarSignal(J_W32_Scheduler *scheduler, J_W32_FiberCondVar *cv);
internal void J_W32_FiberCondVarBroadcast(J_W32_Scheduler *scheduler, J_W32_FiberCondVar *cv);


/* ==================================================
   JOBS
   ================================================== */

internal void J_W32_Push(J_W32_Scheduler *scheduler, const J_W32_Request *request);
internal void J_W32_PushWaitingFiber(J_W32_Scheduler *scheduler, J_W32_Fiber *fiber);

internal void J_W32_Yield(J_W32_Scheduler *scheduler, J_W32_Counter *counter, i32 value);

internal void J_W32_Kick  (J_W32_Scheduler *scheduler, const J_Decl *decl, J_W32_Counter *counter);
internal void J_W32_Batch (J_W32_Scheduler *scheduler, const J_Decl *decls, u32 count, J_W32_Counter *counter);

/*
 * Divides [0, count) into batches of batch_size, and kicks
 * off a basic for-loop job for them all.
 */
internal J_ENTRY_POINT_DEF(J_W32_ParallelForBatchEntry);

internal void J_W32_For(J_W32_Scheduler *scheduler, u32 count, J_EntryForFn *fn, J_Priority priority, u32 batch_size);


/* ==================================================
   SCRATCH
   ================================================== */

internal Arena *J_W32_GetScratch(J_W32_Scheduler *scheduler, Arena * const *conflicts, u32 conflict_count);
```


# Implementation

To execute these jobs, we make it so all threads apart from the master controller thread are added to a "thread pool". Each one of these new "workers" will check if a new job is available. If it isn't it will wait until one is, and if it is then it will pull it from the job queue and execute it by calling the entry point with the params.

TODO


# Potential Improvements

TODO


# Closing Thoughts

Anyway, I'm pretty happy with the end result, and I hope you found this helpful in any way. If you have any questions or ideas on how to improve it then don't hesitate to email me.

Thank you for reading. :)
