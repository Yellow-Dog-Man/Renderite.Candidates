# OGRE-Next
**Repository:** https://github.com/OGRECave/ogre-next

**License:**
- ✅ MIT

## Basic Overview
**Programming language(s):**
- ⚠️ C++
  - 📖 Primary renderer implementation. C/C++ is acceptable under the requirements, but not one of the preferred languages.
- ⚠️ C and Objective-C++
  - 📖 Platform glue where needed.

**Shading language(s):**
- ✅ HLMS
  - 📖 Primary high-level material system.
- ✅ GLSL
- ✅ HLSL
- ✅ MSL
- ✅ GLSL-for-Vulkan / SPIR-V paths

**Rendering paths:**
- ✅ Clustered forward
  - 📖 HLMS PBS and Unlit material systems are present.

**Graphics API's:**
- ✅ Vulkan
- ✅ Metal
- 🛑 Direct3D 12
  - 📖 Direct3D 12 support was not found. A related tracking issue exists at https://github.com/OGRECave/ogre-next/issues/561.
- ⚠️ Direct3D 11
  - 📖 Supported by OGRE-Next, but not an acceptable primary API under the requirements.
- ⚠️ OpenGL 3.3+ / OpenGL ES 2.x/3.x
  - 📖 Supported by OGRE-Next, but not an acceptable primary API under the requirements. Vulkan is also available for newer explicit-API use cases.
- 🗨 Null

**Supported platforms:**
- ✅ Windows 7+
- ✅ Android
- ⚠️ Quest
  - 📖 Android support is present. Quest-specific renderer platform support was not separately confirmed.
- ✅ Linux
- ✅ macOS
- ✅ iOS
- 🗨 Emscripten / Web-oriented builds

**Oldest supported hardware:**
- ✅ DX11/OpenGL 3.3-class desktop hardware for the main renderer
- ⚠️ Android Vulkan-capable hardware
- ⚠️ iOS Metal-capable hardware

**VR API's:**
- 🛑 OpenXR
  - 📖 OpenXR support was not found.
- ⚠️ OpenVR
  - 📖 Single-pass stereo is implemented separately from the OpenVR sample path. An OpenXR implementation could use the existing VR support as reference, but would still require integration work.

**Activity:**
- Rough contributor count: ✅ ~300 listed authors
- Rough user/community size: 🗨 Medium community
  - 📖 Community size is difficult to measure because OGRE and OGRE-Next have a long-lived, diffuse user base. The project has an [OGRE-Next forum](https://forums.ogre3d.org/viewforum.php?f=25) and a [Matrix/Gitter room](https://matrix.to/#/#OGRECave_ogre-next:gitter.im).

## AI Use
How AI is actually used in the project, whether AI contributions are allowed, and whether the renderer is meaningfully human-designed or substantially AI-coded without technical oversight. Note that post-2021 public git history cannot reliably prove that merged code was never AI-assisted. OGRE-Next should not be described as AI-free.

- Observed evidence
  - Git history includes 2026 commits with `Co-Authored-By: Claude ...` trailers, including formatting and renderer/utility fixes (`edd385cad0`, `998c0530c7`, `8ec6166ea3`, `e0126edeb3`).
- Project policy
  - No project-level AI contribution policy was found.
- Evaluation impact
  - OGRE-Next remains a long-running renderer with normal maintainer review, but recent history contains explicit AI co-author markers.

# Existing usage / projects made with this
The OGRE-Next README lists [Yoy Simulators](https://www.yoy.cl/), Skyline Game Engine, Racecraft, Sunset Rangers, and Stunt Rally 3 as projects using OGRE-Next. Yoy Simulators is a notable commercial VR simulator integrator.

# General notes
OGRE-Next is a mature renderer library with strong low-level rendering coverage. The main integration risk is that several systems Resonite currently receives from Unity or Renderite would need to be built around the renderer rather than configured inside it.

## Positive highlights
- MIT licensed.
- Renderer-focused project rather than a full game engine.
- Vulkan and Metal backends are present.
- The renderer is designed around data-oriented performance, cache-friendly entity/node layout, threaded culling, SIMD-friendly data layout, clustered forward rendering, and background texture streaming.
- The renderer has substantial GI, reflection, compute, compositor, HLMS/PBS, instanced stereo, and texture-management infrastructure.

## Potential concerns
- Direct3D 12 support was not found. A related tracking issue exists at https://github.com/OGRECave/ogre-next/issues/561.
- OpenXR support was not found.
- Input, audio, video playback, networking, physics, and other engine-level systems are outside OGRE-Next's core scope.
- Several Resonite-specific workflows would need integration work on top of OGRE-Next, especially video textures, OpenXR, input proxying, camera stacking, and renderer-owned scene submission.
- Motion blur, mesh shaders, and a complete VRS/eye-tracked foveation path were not found.

## Other notes
- OGRE-Next's own README explicitly describes it as a 3D graphics rendering engine, not a full game engine.
- Sample applications use SDL2 for window/input handling.


# New Renderer Requirements
This is the "meat" of the evaluation - going through our list of requirements and checking how well the renderer matches the required features.

> [!IMPORTANT]
> Please read the [README.md](/README.md) on how to contribute and work with this document!

> [!NOTE]
> This is based on the [REQUIREMENTS.md](/REQUIREMENTS.md) document, but with some notes & context removed to keep it cleaner.
> When evaluating features, check the matching features in that document for more context / information!

## VR Rendering
**Required:**
- ⚠️ Allow implementation of dynamic VR/desktop switching
  - 📖 OGRE-Next has compositor workspaces, cameras, visibility masks, and an OpenVR sample. A built-in runtime desktop/VR switching workflow was not found.

**Ideal:**
- ✅ Single-pass stereo rendering
  - 📖 Instanced stereo is implemented in compositor scene passes and HLMS PBS/Unlit shaders.
- ⚠️ Canted displays rendering support (e.g. Pimax)
  - 📖 Per-eye VR view and projection matrices are present. A 2019 OGRE post notes that instanced stereo was designed with per-eye rotation in mind and may work for large-FOV headsets such as Pimax and StarVR, but it had only been tested with Oculus and Vive at that time: https://www.ogre3d.org/2019/09/22/improvements-in-vr-morph-animations-moving-to-github-and-ci#more-4235. Confirmed headset-specific support for canted displays was not found.
- ⚠️ Foveated rendering support
  - 📖 Radial density mask and foveated VR compute shader assets are present. OpenXR foveation, eye-tracked foveation, and VRS-based foveation were not found.

## Render pipeline
**Required:**
- 🗨 General performance on par or better with current Unity renderer
  - 📖 OGRE-Next has strong performance-oriented design indicators: clustered forward rendering, data-oriented scene layout, threaded frustum culling, SIMD-friendly storage, command-buffer style rendering, instanced stereo, and background texture streaming. Resonite parity would still need benchmark scenes.
- ✅ Some level of control over graphics pipeline rendering
  - 📖 The compositor system supports scene, quad, compute, custom, clear, stencil, mipmap, and IBL-related passes.
- ✅ Ability to control the order of rendering and sorting
  - 📖 Render queues, queue groups, queue subgroups, and compositor render queue ranges provide direct ordering control.
  - ⚠️ **Ideal:** Ability to create a "group/batch" of render entities that are rendered at once as a unit and have their own internal sorting order
    - 📖 Render queue groups and subgroups provide similar control. A dedicated grouped render entity with its own internal sort was not found.

- ✅ Stencil buffer support
  - Supported operations must match the currently exposed ones through materials
    - ✅ 8-bit integer (0 to 255)
    - ✅ Comparison modes:
      - ✅ Disabled
      - ✅ Never
        - 📖 OGRE calls this `CMPF_ALWAYS_FAIL`.
      - ✅ Less
      - ✅ Equal
      - ✅ LessOrEqual
        - 📖 OGRE calls this `CMPF_LESS_EQUAL`.
      - ✅ Greater
      - ✅ NotEqual
      - ✅ GreaterOrEqual
        - 📖 OGRE calls this `CMPF_GREATER_EQUAL`.
      - ✅ Always
        - 📖 OGRE calls this `CMPF_ALWAYS_PASS`.
    - ✅ Stencil Operations
      - ✅ Keep
      - ✅ Zero
      - ✅ Replace
      - ✅ IncrementSaturate
        - 📖 OGRE documentation refers to this as `Increment`.
      - ✅ DecrementSaturate
        - 📖 OGRE documentation refers to this as `Decrement`.
      - ✅ Invert
      - ✅ IncrementWrap
      - ✅ DecrementWrap
    - ✅ Read & Write masks
- ✅ LOD support
  - ✅ Per-render switching between render entities depending on relative size on the screen
  - 🛑 Ideally blending support (e.g. through material/shader properties)
  - 📖 Mesh LOD generation and screen-size based LOD selection exist. LOD blending support was not found.
- ✅ Some form of Global Illumination (GI) support
  - 📖 OGRE-Next has substantial GI coverage. The GI documentation lists parallax-corrected cubemaps, per-pixel PCC, irradiance volumes, voxel cone tracing, cascaded image voxel cone tracing, irradiance fields with depth, and instant radiosity: https://ogrecave.github.io/ogre-next/api/latest/_gi_methods.html#GiPCC.
- ✅ GPU instancing
  - ✅ Ideally fully automated from the render entities
  - ✅ Efficiently render large number of entities with the same material & mesh
  - ⚠️ Should have some form of support for varying material properties (e.g. fetching them from a buffer)
  - 📖 OGRE-Next 2.3 documentation describes HLMS auto-instancing and notes that InstanceManager is mainly useful for very large instance counts with the same mesh and material: https://ogrecave.github.io/ogre-next/api/2.3/instancing.html. Current HLMS paths still include instancing support. Per-object data buffers and texture buffers exist, but arbitrary Resonite-style per-instance material variation would need integration work.
- ⚠️ Mirror/Portal rendering
  - ⚠️ Ideally should be as efficient as possible for VR - use single pass rather than two separate renders
  - ✅ More flexible on render method
    - ✅ Skewed matrix with render to texture (current method)
    - ✅ Scissor/Stencil is a potential alternative method
  - 📖 Planar reflections, render-to-texture, custom camera/projection setup, scissor, and stencil support are present. A complete VR-efficient portal/mirror renderer was not found.

**Ideal:**
- ⚠️ HDR display output
  - 📖 HDR rendering and HDR/SMAA samples are present. A full platform HDR display output workflow was not found.
- ✅ Multiple Window support
  - 📖 OGRE has render-window abstractions and compositor workspaces that can target separate render targets. A Resonite-style multi-window workflow would still need integration.
- 🛑 Mesh shaders (with meshlets) pipeline
  - 🛑 Provide fallback for older GPU's
  - 📖 Mesh shader support was not found beyond API extension headers.
- ⚠️ Better alpha sorting/blending handling
  - 📖 Standard transparent sorting and alpha-related material features exist. OIT or a comparable advanced transparency solution was not found.

**Nice to have:**
- 🛑 Forms of static / dynamic occlusion culling
  - 📖 Frustum culling exists. Static/dynamic occlusion culling was not found.
- ⚠️ Ability to re-render the same mesh multiple times with different materials efficiently
  - ⚠️ Perform frustum / occlusion culling just once
  - ⚠️ For skinned meshes - transform vertices just once
  - 📖 Submeshes, multiple material passes, render queue control, and reused mesh resources are available. A full material-stacking path that guarantees one culling and one skinning pass was not found.
- ✅ Reversed floating point depth buffer

## Shader pipeline
**Required:**
- ✅ Shader pipeline must be isolated enough (or made to be isolated) so it can be invoked from our own code at runtime to dynamically compile shaders
  - 📖 HLMS, shader cache entries, custom HLMS pieces, low-level GPU programs, and compute jobs provide runtime shader generation/compilation hooks.
- ⚠️ Compute Shader support
  - ⚠️ Needs to support wave intrinsics
  - 📖 Compute shaders are implemented through HLMS compute jobs and compositor compute passes. Some sample compute shaders use subgroup/vendor-style helpers such as `anyInvocationARB`, but a broad cross-platform wave-intrinsics abstraction was not found.

**Ideal:**
- 🛑 Slang support
- ✅ SPIR-V
  - 📖 The Vulkan path uses GLSL-for-Vulkan and SPIR-V-related tooling.
- ✅ Existing PBR/PBS shaders
  - 📖 HLMS PBS is a core material system.

## Post processing
To match feature parity, the rendering pipeline needs to support the same/similar post processing filters as our current render currently does.

**Required:**
- ✅ Bloom
  - ✅ Must support working with HDR values (above 1.0)
  - 📖 The HDR sample includes a bloom/glare-style post-processing chain over HDR render targets.
- 🛑 Motion Blur
  - 🛑 Motion vector based
  - 🛑 Support camera blur
  - 🛑 Support object blur
  - 🛑 Support skinned mesh blur
  - 🛑 Ideally supported both in VR & desktop, but desktop is sufficient
  - 📖 Motion blur support was not found.
- ✅ Ambient Occlusion
  - 📖 SSAO sample and compositor materials are present.
- ✅ Anti-aliasing
  - ✅ MSAA (given by the rendering path)
  - 📖 SMAA sample and compositor materials are also present.
- ⚠️ VR & single pass support
  - 📖 The compositor can process VR render targets and instanced stereo is available for scene rendering. Individual post effects would need validation in the VR single-pass path.

**Nice to have:**
- ✅ Screen space reflections
  - 📖 A ScreenSpaceReflections API usage sample and compositor setup are present.
- 🛑 Some form of screen space realtime GI
  - 📖 Screen-space realtime GI support was not found.

**Not required:**
  - ✅ Anti-aliasing (these are largely needed for deferred rendering)
  - 🛑 TAA
  - 🛑 FXAA
  - 🛑 CTAA
  - ✅ SMAA
  - ...

## Rendering components / entities
Note that the structure does not need to match the current ones, but the engine needs to have API's and structures so we can replicate functionality of these. It is *not* required that the render specifically has those entities implemented, but rather that it has features that allow them to be implemented.

### Mesh Rendering
**Must have:**
- ✅ Triangle topology
- ✅ Point topology
- ✅ Some form of submesh support
  - ✅ Allow specifying materials for each submesh - the whole mesh is rendered & culled as a unit, but each submesh is its own
- ✅ Shadow rendering support (depending on material support)
  - ✅ Single sided
  - ✅ Dual sided
  - ✅ None
  - ⚠️ Shadow only
  - 📖 Cast-shadow flags and culling controls are available. A simple built-in shadow-only render mode was not found.

### Skinned Mesh Rendering
These requirements are on top of standard mesh rendering.

**Required:**
- ✅ 1-4 bone support
- ⚠️ Blendshape support
  - ✅ Multi-frame support (each blendshape goes through several frames)
  - ⚠️ Positions, Normals & Tangents
  - 📖 OGRE supports pose and morph-style vertex animation. Pose normal handling exists in HLMS properties, but full Resonite-style position/normal/tangent blendshape parity would need validation.
- ✅ GPU accelerated for good performance
  - 📖 HLMS PBS contains shader paths for skeleton and pose animation.

**Nice to have:**
- ⚠️ More than 4 bones
  - 📖 Skeletons can contain many bones. More than four bone weights per vertex was not confirmed.
- ⚠️ Blendshape support
  - 🛑 UV's, Colors
  - ⚠️ Ability to cache rarely changing blendshapes to avoid recomputations
  - ⚠️ Ability to only compute affected vertices & skip rest
  - 📖 Pose animation can represent sparse affected vertices. A dedicated GPU cache path for rarely changing blendshapes was not found.
- 🛑 Dual Quaternion Skinning #487
  - 📖 Dual quaternion math/tests exist, but a dual quaternion skinning path was not found.

### Lights
**Required:**
- Supported types:
  - ✅ Point
  - ✅ Spot
  - ✅ Directional
- Supported features:
  - ✅ Hard & Soft realtime shadows for all types
  - ✅ Multiple instances of each type
  - ⚠️ Light cookies for point & spot lights
    - 📖 IES light profiles are present. Unity-style point/spot cookie texture support was not found.
- ⚠️ Control over lighting falloff
  - 📖 Standard attenuation controls and clustered forward fade ranges exist. Arbitrary user-defined falloff curves were not found.

**Nice to have:**
- 🛑 RGB light cookies
  - 📖 RGB light cookie support was not found.
- ⚠️ Baked shadowmaps
  - 📖 Static shadowmap-related sample assets are present. A complete baked shadowmap workflow was not found.
- ⚠️ Control over realtime shadowmap rendering
  - ⚠️ Ideally the pipeline could render point/spot light shadowmaps once each frame and share them for each camera/view that's rendered that frame
  - 📖 Shadow nodes and compositor control provide scheduling hooks. A confirmed shared-per-frame shadowmap workflow across all cameras was not found.
- ✅ Realtime area/polygonal lights
  - 📖 LTC area-light support is present.

### Cameras
**Required:**
- ✅ Support rendering additional views (other than primary view) into a render texture
- ⚠️ Double buffering support (cameras see the contents of their own render texture from previous frame)
  - 📖 This can be implemented with two render textures and compositor scheduling. A built-in camera double-buffer mode was not found.
- ⚠️ Support camera stacking with render order (depth)
  - ✅ Multiple cameras must be able to render into the same texture
  - ⚠️ Order must be able to be defined (e.g. with depth value)
  - ✅ Must support viewport configuration (rendering to a sub-section of the render texture)
  - ✅ Must support different clear methods (none, depth only, color/skybox)
  - 📖 Compositor workspaces, passes, clear/load/store actions, and viewport/scissor controls are present. A Unity-like camera stack with a simple depth property was not found.

### Reflection probes
**Required:**
- ✅ Reflection probe that affects specular lighting on materials (notably PBS materials)
- ✅ Varying smoothness support stored in mip maps
- ✅ Real time reflection probe rendering
  - ⚠️ Support for time slicing when rendering (doesn't need to be baked in as long as we have control where individual phases of render occur)
- ✅ Baked reflection probes
  - ✅ Using cubemap with mipmaps storing different smoothness levels
- ✅ Rendering reflection probe to cubemap asset
- ✅ Control over when is reflection probe rendered
  - 📖 Cubemap probes, parallax-corrected cubemaps, IBL specular processing, dynamic cubemap rendering, and PCC automation are present. Explicit automatic time slicing was not found, but compositor-level control is available.

## Scene model API
Since FrooxEngine has its own scene model, the ideal state for the renderer is to be minimalistic and hold only data absolutely necessary for the rendering itself. If needed, we can have our own code to hold this data on top of the renderer if it doesn't naturally hold it between frames.

**Required:**
- ✅ Mechanism to filter which entities are rendered for each camera
  - 📖 Visibility masks, render queue ranges, and compositor pass filtering provide per-camera/per-pass filtering mechanisms.

**Ideal:**
- ⚠️ Flat scene description with no transform hierarchy - we submit 4x4 matrices for each entity to be rendered
  - 📖 OGRE-Next uses scene nodes and retained scene objects. A flat submission layer could be built on top, but it is not the native public model.
- ⚠️ Submitted poses are in world space and can be shared between multiple cameras/views
  - 📖 Retained scene transforms can be shared by multiple cameras/workspaces. Direct flat world-space pose submission is not the native workflow.
- ✅ Renderer performs camera frustum culling & sorting

**Possible alternatives:**
 - ⚠️ Camera frustum culling & sorting is performed on engine side & fully sorted list of "render commands / draw calls" is submitted to the renderer for each camera / view
   - 📖 Custom renderables and compositor passes make this plausible, but a clean external sorted draw-command submission API was not found.

**Not viable:**
 - Individual drawcalls and render commands are proxied from the main process
  - This would incur too much overhead for IPC
  - It's possible that the renderer works this way on its side and we write code to hold the simplified scene state, but the scene state needs to be submitted in bulk

## Asset/resource support

### Textures
**Required:**
- Texture types
  - ✅ 2D
  - ✅ Cubemap
  - ✅ 3D
- ✅ API to upload arbitrary texture data to the GPU provided in a byte buffer
  - ✅ Support for uploading texture sub-regions
  - ⚠️ Support for specifying max mip level to currently use
  - ✅ Support for dynamic updates (textures can be updated anytime - important for procedural textures)
  - 📖 `TextureGpu`, `TextureBox`, `StagingTexture`, texture residency, and mip-related APIs are present. A high-level Resonite-style max-active-mip workflow would need integration.
- ✅ Support for common compressed formats (depending on HW)
  - ✅ Uncompressed formats (e.g. RGBA32/ARGB32)
  - ✅ Block compressed formats (BCx, ETCx, ASTC...)

**Ideal:**
- ⚠️ Uploads can be fully done from background threads - engine takes care of any necessary synchronization
  - 📖 File texture loading has background streaming support. Manual staging texture upload still involves render-system synchronization and main-render-thread integration.

### Meshes
**Required:**
- Supported vertex attributes
  - ✅ Positions
  - ✅ Normals
  - ✅ Tangents
  - ✅ UV's
    - ✅ At least 8 channels
    - ✅ 2-4 dimensions per each channel
  - ✅ Colors
  - 📖 Vertex declarations support position, normal, tangent, diffuse/color, and indexed texture-coordinate semantics with float vector formats.
- Supported topologies
  - ✅ Triangles
  - ✅ Points
- ⚠️ Skinned mesh data
  - ✅ Support binding 1-4 bone transforms to each vertex
  - ⚠️ Support blendshape data in some form
    - ⚠️ Positions, normals, tangents
    - ✅ Support more than one frame for blendshape (progression of blendshape goes through multiple frames)
  - 📖 The Mesh API documentation discusses adding bone weights and assigning skeletons to meshes: https://ogrecave.github.io/ogre-next/api/latest/class_ogre_1_1_mesh.html#details. Skeleton and pose/morph animation systems are present. Exact parity for all Resonite blendshape channels would need validation.

### Video Textures
Since Resonite supports video playback and this is handled by the renderer (due to GPU texture resources being updated) so this support will need to be handled by the new renderer as well.

This will most likely have to be implemented - e.g. through doing integration with libVLC.

**Required:**
- ⚠️ Video texture support integrated with the rendering pipeline
  - ⚠️ Ability to control the playback (playing, looping, playback position)
  - 🛑 Ability to select audio track that's decoded
  - 📖 OGRE has an ExternalTextureSource plugin API and documentation references an ffmpeg video plugin. A maintained video playback implementation in the OGRE-Next repo was not found.
- 🛑 Ability to get raw audio data
- 🛑 Support for both local file playback & streaming
  - 🛑 Streaming video files from a web endpoint
  - 🛑 Supporting live video streams (like rtmp)

**Ideal:**
- 🛑 Ideally libVLC integration for maximum compatibility
  - Can be potentially other playback engines, but the format support needs to be similar
- 🛑 HW/GPU decoding support
- 🛑 Specifying separate video & audio stream URL's

## Input handling
Since the renderer will be providing the user a window and interfacing with some of the input devices (e.g. VR), we need to be able to proxy various inputs.

If these are not supported, they should be trivial to implement in Phase 3.

- 📖 Input is outside OGRE-Next's core renderer scope. Community guidance and sample applications use SDL2 for keyboard and mouse input.

**Required:**
- 🛑 Keyboard support (including Unicode)
     - 🛑 Individual key press events
     - 🛑 Type delta
     - 🛑 IME composition
- 🛑 Mouse support
     - 🛑 At least 5 mouse buttons
     - 🛑 Scroll wheel (including horizontal ones)
     - 🛑 Absolute position
     - 🛑 Relative delta
- 🛑 VR input support
     - 🛑 Access state of all controller buttons/elements
     - 🛑 Haptics support
     - 🛑 Access to full skeleton of the hand (for supported controllers)
- 🛑 Touch support
     - 🛑 Multi-touch
     - 🛑 Ideally have access to other properties like pressure

**Nice to have:**
- 🛑 Gamepad support
- 🛑 Pen/Stylus support
     - 🛑 Pressure, tilt etc...

## Other "nice to have"
- 🛑 Spout support
  - 📖 Spout support was not found.
- 🛑 Video encoding support (e.g. FFmpeg/NVENC integration and so on)
  - 📖 Video encoding support was not found.
