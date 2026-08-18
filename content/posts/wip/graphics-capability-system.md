---
title: "Implementing a graphics capability system"
date: 0
categories:
 - Graphics
 - Programming
draft: true
---

Recently I was working on porting my Vulkan Renderer (though now more of almost a game framework), [magpie](https://github.com/kryzp/magpie), to work on my M1 MacBook!

This meant running it through a portability layer such as MoltenVK or KosmicKrisp. So far, I'd been writing my engine with a more cowboy wild-west esque philosophy where I mostly just threw stuff together based on whatever my current computer supported, with the assumption I'll handle capabilities later.

Well, now that time had come, as my computer didn't support hardware raytracing features I was using to experiment with irradiance probe generation. I didn't wanna just cut that out, so I instead decided I wanted to create some form of capability system, where my top level rendering code could request generic features from the graphics backend, and mark them as required or optional.

I couldn't just straight up pass Vulkan extension requirements down, since I try to generally keep direct Vulkan dependencies isolated to the graphics module (though, I don't entirely stick to that. For example, Vulkan enums are allowed to be used outside of it simply because I'm lazy and I'm not going to write an entire enum translation system for no reason).

Therefore, I opted to design a system where I had "features" and "capabilities". The graphics backend could outline the relationship between a "feature" (a front-facing concept such as raytracing) and the internal capabilities that back it (raytracing pipeline, acceleration structures, ray query, ...). Capabilities have to be abstracted as some Vulkan extensions are dependent on OTHER extensions, adding to the mess.

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

function void InitGraphicsModule(..., FeatureRequest *requested_features, u32 feature_count);
```

Internally, the capabilities system is pretty much this:
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

function b32 CapabilitiesExtensionListContains(const char **list,
											   u32 count,
											   const char *name);

function b32 CapabilitiesExtensionAvailable(VkExtensionProperties *available,
											u32 available_count,
											const char *name);

function Capabilities CapabilitiesQuery(VkPhysicalDevice physical_device,
										VkPhysicalDeviceFeatures2 *out_features2,
										LOChannel log_channel);

function ResolvedFeatures FeaturesResolve(Arena *arena,
										  VkPhysicalDevice physical_device,
										  Capabilities detected,
										  const Requirements *requirements,
										  LOChannel log_channel);

function u32 FeaturesScore(Features enabled, const Requirements *requirements);
```

Finally the system can be used like so:

```c
// graphics module init, take in requested_features + feature_count as parameters from a higher level module

static const FeatureDef feature_defs[] = {
	[FeatureType_RayTracing] = {
		.name = "raytracing",
		.capability_count = 3,
		.capabilities[0] = CapabilityType_RayTracingPipeline,
		.capabilities[1] = CapabilityType_AccelerationStructure,
		.capabilities[2] = CapabilityType_RayQuery
	}
};

// extensions required by the module internally
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
  
G_Requirements requirements = {0};
requirements.base_extension_count = ArraySize(base_extensions);
requirements.base_extensions = base_extensions;
requirements.feature_count = feature_count;
requirements.features = features;

// later when evaluating physical devices...

VkPhysicalDeviceFeatures2 features = {0};

Capabilities detected_caps = CapabilitiesQuery(p_device, &features, log_channel);
ResolvedFeatures resolved_features = FeaturesResolve(arena, p_device, detected_caps, &requirements, log_channel);

// query them for whatever, make your decision...

```
