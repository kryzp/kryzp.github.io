---
title: "Simplifying Vulkan Synchronisation"
date: 2026-08-19
categories:
  - Graphics
  - Programming
---

This is just a quick post to show you that synchronisation in Vulkan doesn't have to be a major pain in the backside, with a little thing called (drumroll...) **timeline semaphores**!

The basic idea is that we introduce an abstraction over a point in time: a `TimelinePoint` (yes, very creative, I know).

Whenever you submit work to a queue through our backend, the submission returns a timeline point representing a point in the future. We can then continue doing whatever work we need to do, and when we eventually need to use a resource that the GPU is still working on, we can ask the backend to wait until that timeline point has been reached.

It should be easier to explain what I mean if I just show you the code directly:

```c
typedef struct TimelinePoint TimelinePoint;
struct TimelinePoint
{
	VkSemaphore semaphore;
	u64 value;
};

typedef struct TimelineSemaphore TimelineSemaphore;
struct TimelineSemaphore
{
	VkSemaphore vk_handle;
	u64 last_submitted_value;
};
```

We can then introduce an API for working with these semaphores. Essentially, all we need is some way to reserve a new timeline value and wait for a particular value to be reached (obviously, rename these functions as you see fit):

```c
TimelinePoint SignalTimelineSemaphore(TimelineSemaphore *semaphore)
{
	semaphore->last_submitted_value++;

	TimelinePoint p = {
		semaphore->vk_handle,
		semaphore->last_submitted_value
	};

	return p;
}

void WaitUntil(TimelinePoint point)
{
	if (point.value == 0)
		return;

	VkSemaphoreWaitInfo wait_info = {0};
	wait_info.sType = VK_STRUCTURE_TYPE_SEMAPHORE_WAIT_INFO;
	wait_info.semaphoreCount = 1;
	wait_info.pSemaphores = &point.semaphore;
	wait_info.pValues = &point.value;

	VK_CHECK(vkWaitSemaphores(GetVkDeviceSomehow(),
							  &wait_info,
							  UINT64_MAX),
			   "Failed to wait on timeline semaphore");
}
```

`WaitUntil()` is a CPU-side wait: the calling thread will block until the GPU has reached the requested timeline value.

Of course, we also need some way to actually create these semaphores:

```c
TimelineSemaphore CreateTimelineSemaphore(u64 value)
{
	VkSemaphoreTypeCreateInfo timeline_type_create_info = {0};
	timeline_type_create_info.sType = VK_STRUCTURE_TYPE_SEMAPHORE_TYPE_CREATE_INFO;
	timeline_type_create_info.semaphoreType = VK_SEMAPHORE_TYPE_TIMELINE;
	timeline_type_create_info.initialValue = value;

	VkSemaphoreCreateInfo timeline_semaphore_create_info = {0};
	timeline_semaphore_create_info.sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO;
	timeline_semaphore_create_info.pNext = &timeline_type_create_info;

	VkSemaphore vk_semaphore = VK_NULL_HANDLE;

	VK_CHECK(vkCreateSemaphore(GetVkDeviceSomehow(),
							   &timeline_semaphore_create_info,
							   NULL,
							   &vk_semaphore),
			   "Failed to create timeline semaphore.");

	TimelineSemaphore semaphore = {0};
	semaphore.vk_handle = vk_semaphore;
	semaphore.last_submitted_value = value;

	return semaphore;
}

void DestroyTimelineSemaphore(const TimelineSemaphore *semaphore)
{
	vkDestroySemaphore(GetVkDeviceSomehow(),
					   semaphore->vk_handle,
					   NULL);
}
```

Finally, we can introduce a couple of utilities that might be useful:

```c
TimelinePoint LastSignalledPoint(const TimelineSemaphore *semaphore)
{
	TimelinePoint p = {
		semaphore->vk_handle,
		semaphore->last_submitted_value
	};

	return p;
}

u64 TimelineSemaphoreGPUCounterValue(const TimelineSemaphore *semaphore)
{
	u64 result = 0;

	vkGetSemaphoreCounterValue(GetVkDeviceSomehow(),
							   semaphore->vk_handle,
							   &result);

	return result;
}
```

And that's pretty much our entire API!

Now let's talk about how to actually use these semaphores to simplify our Vulkan synchronisation code.


## Swapchain Synchronisation

There is one area where timeline semaphores don't replace binary semaphores: synchronisation with the presentation engine.

Your `render_finished_semaphore` and `image_available_semaphore` (or their equivalents) should remain binary semaphores.

As a side note, I really recommend splitting your swapchain frame data from your device frame-in-flight data. They are not the same thing!

Your swapchain's number of images is determined by the presentation system and the surface capabilities, while the number of device frames in flight is something you choose based on how much work you want to have queued up, typically two or three.

So, something like this:

```c
typedef struct SwapchainFrame SwapchainFrame;
struct SwapchainFrame
{
	// ...

	// Signaled when rendering has finished and the image
	// is ready to be presented.
	VkSemaphore render_finished_semaphore;
};

typedef struct Swapchain Swapchain;
struct Swapchain
{
	// ...

	u32 current_frame_index;
	u32 frame_count;
	SwapchainFrame *frames;
};

typedef struct DeviceFrame DeviceFrame;
struct DeviceFrame
{
	// ...

	// Signaled by the presentation engine when an image
	// becomes available for rendering.
	VkSemaphore image_available_semaphore;
};

typedef struct Device Device;
struct Device
{
	// ...

	u32 current_frame_in_flight_index;
	DeviceFrame frames_in_flight[MAX_FRAMES_IN_FLIGHT];
};
```

Now we can get to the interesting part: actually using timeline semaphores for GPU work.


## Submitting Work

The real power of timeline semaphores is that a single semaphore can represent an entire sequence of submissions. Instead of creating a binary semaphore for every dependency, we can simply refer to a particular point in the timeline.

This makes it much easier to scale our synchronisation system as we add more queues and more complicated dependencies.

For example, we can give the graphics queue its own timeline semaphore:

```c
typedef struct Device Device;
struct Device
{
	// ...

	TimelineSemaphore graphics_semaphore;
};
```

We can then build our front-facing API around timeline points:

```c
TimelinePoint SubmitToGraphics(const CmdBuffer *cmd /*, other params, e.g. extra semaphores */)
{
	TimelineSemaphore *graphics_semaphore = GetDeviceGraphicsSemaphoreSomehow();

	TimelinePoint timeline_point = SignalTimelineSemaphore(graphics_semaphore);

	VkSemaphoreSubmitInfo timeline_signal_info = {0};
	timeline_signal_info.sType = VK_STRUCTURE_TYPE_SEMAPHORE_SUBMIT_INFO;
	timeline_signal_info.semaphore = timeline_point.semaphore;
	timeline_signal_info.value = timeline_point.value;
	timeline_signal_info.stageMask = VK_PIPELINE_STAGE_2_ALL_COMMANDS_BIT;

	VkCommandBufferSubmitInfo buffer_info = {0};
	buffer_info.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_SUBMIT_INFO;
	buffer_info.deviceMask = 1;
	buffer_info.commandBuffer = cmd->vk_handle;

	// Call vkQueueSubmit2(...)...

	return timeline_point;
}
```

Now using our API looks something like this:

```c
CmdBuffer cmd = GetCommandBuffer();

// Write some commands...

TimelinePoint point = SubmitToGraphics(&cmd /*, ... */);

// Do some CPU work...

WaitUntil(point);
```

The important part here is that the submission gives us a concrete point in the queue's timeline that we can refer to later.

We don't need to create a new semaphore for every dependency. We don't need to keep track of a collection of binary semaphores and fences. We can simply say:

> "I need the graphics queue to have reached this point before I continue."


## Frames in Flight

We can also use the exact same mechanism to track when a device frame has finished.

Our `DeviceFrame` can store the timeline point corresponding to the submission that completes that frame:

```c
typedef struct DeviceFrame DeviceFrame;
struct DeviceFrame
{
	// ...

	TimelinePoint completion_point;
};
```

When we start reusing a device frame, we simply wait for its completion point:

```c
WaitUntil(frame->completion_point);
```

Internally, this is essentially the same mechanism we've already been using. We reserve a timeline value, submit work that signals that value, and store the resulting `TimelinePoint` alongside whatever resource or frame we're tracking.

This gives us a very simple rule:

**If something is still being used by the GPU, keep the timeline point for the submission that uses it. Before reusing that thing, wait for that point.**

And that's pretty much all there is to it.

I've had to heavily simplify some of the code here to make it easier to read, so the full implementation is available here:

- [`semaphore.h`](https://github.com/kryzp/magpie/blob/master/src/graphics/graphics_semaphore.h)
- [`semaphore.c`](https://github.com/kryzp/magpie/blob/master/src/graphics/graphics_semaphore.c)
- [`swapchain.h`](https://github.com/kryzp/magpie/blob/master/src/graphics/graphics_swapchain.h)
- [`device.h`](https://github.com/kryzp/magpie/blob/master/src/graphics/graphics_device.h)
- [`device.c`](https://github.com/kryzp/magpie/blob/master/src/graphics/graphics_device.c)

Hopefully this makes Vulkan synchronisation feel a little less intimidating.

Thank you for reading! If you have any questions, or if you think I've made any mistakes, don't hesitate to email me.
