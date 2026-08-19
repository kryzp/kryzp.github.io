---
title: "Implementing a Vulkan Hardware Capability System"
date: 2026-08-19
categories:
 - Graphics
 - Programming
---

Recently I was working on porting my Vulkan Renderer (though now more of almost a game framework), [magpie](https://github.com/kryzp/magpie), to work on my M1 MacBook!

This meant running it through a portability layer such as MoltenVK or KosmicKrisp. So far, I'd been writing my engine with a more wild-west esque philosophy where I mostly just threw stuff together based on whatever my current computer supported, with the assumption I'll handle capabilities later.

Well, now that time had come, as my computer didn't support hardware raytracing features I was using to experiment with irradiance probe generation. I didn't wanna just cut that out, so I instead decided I wanted to create some form of capability system, where my top level rendering code could request generic features from the graphics backend, and mark them as required or optional.

I couldn't just straight up pass Vulkan extension requirements down, since I try to generally keep direct Vulkan dependencies isolated to the graphics module (though, I don't entirely stick to that. For example, Vulkan enums are allowed to be used outside of it simply because I'm lazy and I'm not going to write an entire enum translation system for no reason). I needed some way to generalise hardware "features".

Therefore, I opted to design a system where I had "features" and "capabilities". The graphics backend could outline the relationship between a "feature" (a front-facing concept such as raytracing) and the internal capabilities that back it (raytracing pipeline, acceleration structures, ray query, ...).

Initially, I thought we could just directly link features with a list of extensions, however I ran into a speedbump pretty quickly: in Vulkan, an extension being present and a feature actually being usable are two separate questions.

You can query `vkEnumerateDeviceExtensionProperties` and see `VK_KHR_ray_tracing_pipeline` sitting right there in the list, and still have no ray tracing, because the extension only gives you the types, the actual capability is gated behind a boolean in a `VkPhysicalDeviceRayTracingPipelineFeaturesKHR` struct that you have to chain into `vkGetPhysicalDeviceFeatures2` and check separately.

So: features are what I actually want to turn on for rendering, capabilities are the hardware checks, and extensions are at the very bottom of the "tree".


# API

Front-facing code:
```c
typedef enum FeatureTier
{
	FeatureTier_Unused,
	FeatureTier_Optional,
	FeatureTier_Required,
	FeatureTier_COUNT
}
FeatureTier;

typedef enum FeatureType
{
	FeatureType_RayTracing,
	FeatureType_COUNT
}
FeatureType;

typedef struct FeatureRequest
{
	FeatureType type;
	FeatureTier tier;
}
FeatureRequest;

void InitGraphicsBackend(..., FeatureRequest *requested_features, u32 feature_count);
```

Internally, the capabilities system is pretty much this, note that I'm using some unique things like `LOG_Channel` and `Arena`. Don't worry about these, the `LOG_Channel` is just an identifier I use to organise debug logging which you can safely ignore, and the `Arena` is just an allocation system. I'm sure you could easily adapt this to whatever systems you use internally.

```c
typedef enum CapabilityType
{
	CapabilityType_RayTracingPipeline,
	CapabilityType_AccelerationStructure,
	CapabilityType_RayQuery,
	CapabilityType_COUNT
}
CapabilityType;

typedef struct FeatureDef FeatureDef;
struct FeatureDef
{
	const char *name;
	u32 capability_count;
	CapabilityType capabilities[8];
};

typedef struct Feature Feature;
struct Feature
{
	FeatureType type;
	FeatureTier tier;
	FeatureDef def;
};

typedef struct Requirements Requirements;
struct Requirements
{
	b32 base_extension_count;
	const char **base_extensions;
	u32 feature_count;
	Feature *features;
};

typedef struct Capabilities Capabilities;
struct Capabilities
{
	b32 set[CapabilityType_COUNT];
};

typedef struct Features Features;
struct Features
{
	b32 set[FeatureType_COUNT];
};

typedef struct ResolvedFeatures ResolvedFeatures;
struct ResolvedFeatures
{
	Features enabled;
	b32 meets_requirements;
	const char **extension_names;
	u32 extension_count;
};

b32 CapabilitiesExtensionListContains(const char **list,
									  u32 count,
									  const char *name);

b32 CapabilitiesExtensionAvailable(VkExtensionProperties *available,
								   u32 available_count,
								   const char *name);

Capabilities CapabilitiesQuery(VkPhysicalDevice physical_device,
							   VkPhysicalDeviceFeatures2 *out_features2,
							   LOG_Channel log_channel);

ResolvedFeatures FeaturesResolve(Arena *arena,
								 VkPhysicalDevice physical_device,
								 Capabilities detected,
								 const Requirements *requirements,
								 LOG_Channel log_channel);
```


# Usage

Finally the system can be used like so:

```c
// graphics module init, take in requested_features + feature_count as parameters from a higher level module...

static const FeatureDef feature_defs[] = {
	// here, the internal graphics abstraction lays out the relation
	// between front-facing features and internal capabilities.
	[FeatureType_RayTracing] = {
		.name = "raytracing",
		.capability_count = 3,
		.capabilities[0] = CapabilityType_RayTracingPipeline,
		.capabilities[1] = CapabilityType_AccelerationStructure,
		.capabilities[2] = CapabilityType_RayQuery
	}
};

// extensions required by the graphics abstraction internally
static const char *base_extensions[] = {
	VK_KHR_SWAPCHAIN_EXTENSION_NAME,
	VK_KHR_SHADER_NON_SEMANTIC_INFO_EXTENSION_NAME,
	VK_KHR_MAINTENANCE_5_EXTENSION_NAME,
	VK_KHR_MAINTENANCE_6_EXTENSION_NAME,
};

Feature *features = ArenaPushArray(scratch.arena, Feature, feature_count);

for (u32 i = 0; i < feature_count; i++)
{
	features[i].def = feature_defs[requested_features[i].type];
	features[i].type = requested_features[i].type;
	features[i].tier = requested_features[i].tier;
}
  
Requirements requirements = {0};
requirements.base_extension_count = ArraySize(base_extensions);
requirements.base_extensions = base_extensions;
requirements.feature_count = feature_count;
requirements.features = features;

// later when evaluating physical devices...

VkPhysicalDeviceFeatures2 features = {0};

Capabilities detected_caps = CapabilitiesQuery(current_physical_device_being_evaluated, &features, log_channel);
ResolvedFeatures resolved_features = FeaturesResolve(arena, current_physical_device_being_evaluated, detected_caps, &requirements, log_channel);

// query them for whatever, make your decision...
```

Depending on which physical device you choose, you can store the resolved features and query them later:

```c
b32 IsFeatureEnabled(FeatureType type)
{
	return GetPhysicalDeviceResolvedFeatures()->enabled.set[type];
}
```

```c
VkDeviceCreateInfo device_create_info = {0};
device_create_info.enabledExtensionCount = GetPhysicalDeviceResolvedFeatures()->extension_count;
device_create_info.ppEnabledExtensionNames = GetPhysicalDeviceResolvedFeatures()->extension_names;
// ...

if (IsFeatureEnabled(FeatureType_RayTracing))
{
	// add raytracing features to device create info list...
}
```


# Implementation

This is just the actual internal implementation of the capabilities system:

```c
typedef struct CapabilityInfo CapabilityInfo;
struct CapabilityInfo
{
	const char *name;
	u32 extension_count;
	const char *extensions[4];
};

const CapabilityInfo g_caps_infos[] = {
	[CapabilityType_RayTracingPipeline] = {
		.name = "ray_tracing_pipeline",
		.extension_count = 3,
		.extensions = {
			VK_KHR_RAY_TRACING_PIPELINE_EXTENSION_NAME,
			VK_KHR_ACCELERATION_STRUCTURE_EXTENSION_NAME,
			VK_KHR_DEFERRED_HOST_OPERATIONS_EXTENSION_NAME,
		},
	},
	[CapabilityType_AccelerationStructure] = {
		.name = "acceleration_structure",
		.extension_count = 2,
		.extensions = {
			VK_KHR_ACCELERATION_STRUCTURE_EXTENSION_NAME,
			VK_KHR_DEFERRED_HOST_OPERATIONS_EXTENSION_NAME,
		},
	},
	[CapabilityType_RayQuery] = {
		.name = "ray_query",
		.extension_count = 3,
		.extensions = {
			VK_KHR_RAY_QUERY_EXTENSION_NAME,
			VK_KHR_ACCELERATION_STRUCTURE_EXTENSION_NAME,
			VK_KHR_DEFERRED_HOST_OPERATIONS_EXTENSION_NAME,
		},
	}
};

b32 CapabilitiesExtensionListContains(const char **list,
									  u32 count,
									  const char *name)
{
	for (u32 i = 0; i < count; i++)
	{
		if (CStrCompare(list[i], name) == 0)
			return true;
	}
	
	return false;
}


b32 CapabilitiesExtensionAvailable(VkExtensionProperties *available,
								   u32 available_count,
								   const char *name)
{
	for (u32 i = 0; i < available_count; i++)
	{
		if (CStrCompare(available[i].extensionName, name) == 0)
			return true;
	}

	return false;
}

Capabilities CapabilitiesQuery(VkPhysicalDevice physical_device,
							   VkPhysicalDeviceFeatures2 *out_features2,
							   LOG_Channel log_channel)
{
	VkPhysicalDeviceRayTracingPipelineFeaturesKHR rt_pipeline_query = {0};
	rt_pipeline_query.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_RAY_TRACING_PIPELINE_FEATURES_KHR;

	VkPhysicalDeviceAccelerationStructureFeaturesKHR accel_query = {0};
	accel_query.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_ACCELERATION_STRUCTURE_FEATURES_KHR;
	accel_query.pNext = &rt_pipeline_query;

	VkPhysicalDeviceRayQueryFeaturesKHR ray_query_query = {0};
	ray_query_query.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_RAY_QUERY_FEATURES_KHR;
	ray_query_query.pNext = &accel_query;

	out_features2->sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_FEATURES_2;
	out_features2->pNext = &ray_query_query;

	vkGetPhysicalDeviceFeatures2(physical_device, out_features2);

	Capabilities result = {0};
	result.set[CapabilityType_RayTracingPipeline]    = rt_pipeline_query.rayTracingPipeline;
	result.set[CapabilityType_AccelerationStructure] = accel_query.accelerationStructure;
	result.set[CapabilityType_RayQuery]              = ray_query_query.rayQuery;
	
	DebugLogD(log_channel, "Detected Capabilities:");
	
	for (u32 i = 0; i < ArraySize(result.set); i++)
		DebugLogD(log_channel, " %s=%d", g_caps_infos[i].name, result.set[i]);
	
	return result;
}

ResolvedFeatures FeaturesResolve(Arena *arena,
								 VkPhysicalDevice physical_device,
								 Capabilities detected,
								 const Requirements *requirements,
								 LOG_Channel log_channel)
{
	ResolvedFeatures result = {0};
	result.meets_requirements = true;
	
	u32 available_count = 0;
	vkEnumerateDeviceExtensionProperties(physical_device, NULL, &available_count, NULL);
	
	ScratchArena scratch = ScratchBegin(&arena, 1);
	VkExtensionProperties *available = ArenaPushArray(scratch.arena, VkExtensionProperties, available_count);
	vkEnumerateDeviceExtensionProperties(physical_device, NULL, &available_count, available);
	
	u32 max_extensions = requirements->base_extension_count;
 
	for (u32 i = 0; i < requirements->feature_count; i++)
	{
		Feature *f = &requirements->features[i];
 
		for (u32 j = 0; j < f->def.capability_count; j++)
			max_extensions += g_caps_infos[f->def.capabilities[j]].extension_count;
	}
	
	const char **extensions = ArenaPushArray(arena, const char *, max_extensions);
	u32 extension_count = 0;
	
	for (u32 i = 0; i < requirements->base_extension_count; i++)
	{
		const char *name = requirements->base_extensions[i];
		
		if (CapabilitiesExtensionAvailable(available, available_count, name))
		{
			extensions[extension_count++] = name;
		}
		else
		{
			result.meets_requirements = false;
			DebugLogE(log_channel, "Missing mandatory base extension: %s", name);
		}
	}
	
	for (u32 i = 0; i < requirements->feature_count; i++)
	{
		Feature *f = &requirements->features[i];
		
		if (f->tier == FeatureTier_Unused)
			continue;
		
		b32 feature_supported = true;
		
		for (u32 j = 0; j < f->def.capability_count && feature_supported; j++)
		{
			CapabilityType cap = f->def.capabilities[j];
			const CapabilityInfo *info = &g_caps_infos[cap];
			
			if (!detected.set[cap])
			{
				feature_supported = false;
				break;
			}
			
			for (u32 k = 0; k < info->extension_count; k++)
			{
				if (!CapabilitiesExtensionAvailable(available, available_count, info->extensions[k]))
				{
					feature_supported = false;
					break;
				}
			}
		}
		
		if (feature_supported)
		{
			result.enabled.set[f->type] = true;
			
			for (u32 j = 0; j < f->def.capability_count; j++)
			{
				CapabilityType cap = f->def.capabilities[j];
				const CapabilityInfo *info = &g_caps_infos[cap];
				
				for (u32 k = 0; k < info->extension_count; k++)
				{
					const char *name = info->extensions[k];
					
					if (!CapabilitiesExtensionListContains(extensions, extension_count, name))
						extensions[extension_count++] = name;
				}
			}
			
			DebugLogD(log_channel, "Feature \"%s\" is supported by this device!", f->def.name);
		}
		else
		{
			if (f->tier == FeatureTier_Required)
			{
				result.meets_requirements = false;
				DebugLogE(log_channel, "Feature \"%s\" is required but unsupported by this device.", f->def.name);
			}
			else
			{
				DebugLogW(log_channel, "Feature \"%s\" is optional and unsupported by this device.", f->def.name);
			}
		}
	}
	
	result.extension_names = extensions;
	result.extension_count = extension_count;
	
	ScratchRelease(&scratch);
	return result;
}
```


# Fin.

I've had to rename / simplify the code to make it easier to understand, so if you wanna see it in action it's found in the `/graphics/` module of my engine, [magpie](https://github.com/kryzp/magpie).

Thank you for reading!
