# The Forge Framework
**Repository:** https://github.com/confettifx/the-forge

- 📖Repo is mostly very technical, more information can be found on their [webpage](https://theforge.dev/products/the-forge/)
- 📖 Additional product-level information is available on [theforge.dev](https://theforge.dev/products/the-forge/).

**License:** ⚠️ Apache 2.0

- 📖 Apache is an acceptable permissive license under the requirements, but MIT is ideal.

## Basic Overview
**Programming language(s):**

- ⚠️ C/C++
  - 📖 C/C++ is acceptable under the requirements.
  - 📖 Objective-C/Objective-C++, Java, Python, Lua, and platform build scripts are also present for Apple, Android, tooling, scripting, and examples.

**Shading language(s):**

- ⚠️ Forge Shading Language (FSL), an HLSL-style shading language used by The Forge tooling.
  - 📖 Other common shading languages are acceptable, but Slang is listed as ideal.
- ⚠️ HLSL-style source and backend shader generation.
- ✅ SPIR-V output for Vulkan shader paths.
- ⚠️ Metal and GLSL backend shader paths are handled through the FSL/tooling pipeline rather than authored as the primary source language.
- 🛑 Slang support

**Rendering paths:**

- ⚠️ Forward++
  - 📖They mention this a lot and it's a bit unclear what it means. It seems to be 3D tiled like clustered forward, but needs more research.
  - 📖 Clustered local lighting is present in the visibility buffer examples, but a general high-level Forward++ renderer API was not found.
- ⚠️ Visibility Buffer / Triangle Visibility Buffer.
  - 📖 The public examples include `Visibility_Buffer`, `Visibility_Buffer2`, triangle filtering, programmable MSAA, indirect drawing, and clustered light assignment shaders. This is a low-level renderer framework rather than a turnkey Resonite-style render queue.
- ⚠️ Ray tracing examples.
  - 📖 DXR/Vulkan ray tracing examples exist, but advanced RTX GI middleware is described in the README as not publicly available.

**Graphics API's:**

- ✅ Vulkan
  - 📖 Vulkan is used for Linux/Steam Deck, Android, Quest, and some platform targets. The README notes that Windows Vulkan was removed from the official Windows runtime path.
- ⚠️ Metal
  - 📖 Metal is acceptable for macOS/iOS support, but Vulkan is listed as the ideal graphics API.
- ⚠️ DirectX 12
  - 📖 DirectX 12 is acceptable on Windows when abstracted well enough, but Vulkan is listed as the ideal graphics API.
- 🛑 DirectX 11
  - 📖 The README states that DirectX 11 support was removed.

**Supported platforms:**

- ✅ Windows 10+
- ✅ Linux
  - 📖They target the Steam Deck primarily, but SteamOS is Linux, so Linux
- ✅ macOS
- ✅ Android
- ✅ Quest
  - 📖Explicitly called out in repo, so may have special support.
- ✅ iOS
- ⚠️ PS(4/5), Xbox (One/Series), and Switch
  - 📖Also supports PS(4/5), Xbox (One/Series), and Switch, for any potential wild future endeavours on console platforms.
  - 📖 Public access requires accredited developers and a commercial license.

**Oldest supported hardware:** ⚠️ The README lists modern API floors such as DirectX 12, Vulkan 1.1, Metal-capable Apple platforms, and iOS A9-era unit test targets. A single exact oldest supported GPU for the whole framework was not found.

**VR API's:** 

- ✅ OpenXR

**Activity:**
- Rough contributor count: Large industry-backed project.
  - 📖 8 main contributors, many small PRs (~150 PRs, unknown how many are duplicate contributors)
- Rough user/community size: Large industry-backed project.
  - 📖 VERY large - this is a framework that has been implemented in several AAA titles. However, the Discord is behind an approval gate, and I have never been able to get access, so I don't know if they're just very picky and only let a select amount of developers in.

## AI Use
The Forge is a long-running, human-designed C/C++ graphics framework maintained by The Forge Interactive. No project AI-use policy, AI-generated contribution policy, or evidence of unreviewed AI-generated renderer work was found in the repository source.

# Existing usage / projects made with this
The Forge has been implemented in a large amount of AAA releases. These include, but are not limited to:

- The official Quest port of Star Wars: Bounty Hunter
  - 📖 The README describes Star Wars: Bounty Hunter port work, including Quest-related work.
- Call of Duty: Warzone Mobile
- Creation Engine (Starfield, TES6, etc.)
  - 📖 The public README specifically discusses Starfield-related renderer work and describes Creation Engine work at a high level.
- No Man's Sky
  - 📖 The README describes macOS and iOS work.
- Forza Motorsport
- Hades
  - 📖 The README describes Supergiant/Hades engine work.
- Skydance's Behemoth optimization work

These are all proprietary games that don't really talk about their middleware usage, but The Forge Framework is essentially a "pick what you need" framework for putting together a AAA-quality renderer.

The public repository generally describes these relationships at a high level rather than exposing title-specific integration code.

# General notes
Anything noteworthy that's not related to the any of the requirements directly should be added to this section.

The Forge is best understood as a production-oriented rendering framework and platform abstraction layer, not as a full game engine. It provides modern graphics API backends, resource loading, shader tooling, input/windowing layers, math, profiling, Dear ImGui integration, Lua scripting support, Ozz animation integration, and example renderers.

## Positive highlights

This is a massively adopted AAA-grade renderer and could open up a large amount of rendering possibilities for us in the future, especially as this is backed by an entity that works with AAA devs on optimization.

- The project has production use in shipped commercial games.
- It targets modern low-level graphics APIs and avoids legacy OpenGL/DirectX 11 renderer paths.
- The source contains substantial examples for visibility-buffer rendering, GPU-driven rendering, ray tracing, skinning, particles, wave intrinsics, screen-space reflections, animation, and platform integration.
- Quest/OpenXR support is present, including multiview and foveation-related code paths.

## Potential concerns

There is nearly no public-facing documentation and the official Discord server seems really hard to get into. They have a pre-screening application thing that uses the Discord awaiting approval thing, and I have sat there for over a month before with no response or reachout.

- Public documentation is sparse compared with engine-style projects, and much of the useful information is embedded in source examples.
- The framework is low-level. A Resonite integration would likely need a substantial scene submission layer, material layer, render queue, video texture layer, and editor/debug tooling on top.
- Some advanced features described in the README, especially RTX GI middleware and console platform support, are not part of the public open-source package.
- The public Windows runtime path is DirectX 12-focused. Vulkan remains important elsewhere, but the README notes that Windows Vulkan was removed from the official runtime path.

## Other notes

- The repository includes a custom ECS integration through Flecs, but the renderer itself does not appear to impose a complete scene graph.
- The public examples are valuable for validating capabilities, but many features are exposed as low-level graphics primitives rather than ready-made engine components.

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
  - 📖 OpenXR initialization, swapchain creation, and normal desktop window/swapchain paths are present. A built-in runtime feature for switching a running app between VR and desktop was not found.

**Ideal:**
- ✅ Single-pass stereo rendering
  - 📖 VR multiview texture flags and Vulkan multiview render pass support are present.
- ⚠️ Canted displays rendering support (e.g. Pimax)
  - 📖 OpenXR per-view pose and projection handling is present, which is the correct basis for canted displays. A Pimax-specific validation path was not found.
- ✅ Foveated rendering support
  - 📖 OpenXR foveation functions and VR foveated texture flags are present.

## Render pipeline
**Required:**
- ⚠️ General performance on par or better with current Unity renderer
  - 📖 The project has strong production-use evidence and modern GPU-driven examples.
- ✅ Some level of control over graphics pipeline rendering
  - 📖 The API exposes explicit render targets, root signatures, descriptor sets, pipeline descriptions, command buffers, barriers, indirect execution, viewports, scissors, and pass control.
- ⚠️ Ability to control the order of rendering and sorting
  - 📖 The framework gives application code direct command-buffer and pass ordering control. A high-level render queue with Unity-style sorting rules was not found.
  - ⚠️ **Ideal:** Ability to create a “group/batch” of render entities that are rendered at once as an unit and have their own internal sorting order
    - 📖 The visibility-buffer and indirect-draw examples show batched GPU-driven rendering, but a reusable high-level entity group API was not found.
- ✅ Stencil buffer support
  - Supported operations must match the currently exposed ones through materials
    - ✅ 8 byte integer (0 to 255)
      - 📖 The source exposes the usual 8-bit stencil masks and operations through graphics API depth/stencil state.
    - ✅ Comparison modes:
      - ✅ Disabled
      - ✅ Never
      - ✅ Less
      - ✅ Equal
      - ✅ LessOrEqual
      - ✅ Greater
      - ✅ NotEqual
      - ✅ GreaterOrEqual
      - ✅ Always
    - ✅ Stencil Operations
      - ✅ Keep
      - ✅ Zero
      - ✅ Replace
      - ✅ IncrementSaturate
      - ✅ DecrementSaturate
      - ✅ Invert
      - ✅ IncrementWrap
      - ✅ DecrementWrap
    - ✅ Read & Write masks
- 🛑 LOD support
  - 🛑 Per-render switching between render entities depending on relative size on the screen
  - 🛑 Ideally blending support (e.g. through material/shader properties)
- ⚠️ Some form of Global Illumination (GI) support
  - 📖 Image-based lighting and PBR environment workflows are present in examples. The README describes advanced RTX GI middleware, but states that it is not publicly available.
- ⚠️ GPU instancing
  - 🛑 Ideally fully automated from the render entities
  - ✅ Efficiently render large number of entities with the same material & mesh
    - 📖 Instanced and indirect draw paths are present.
  - ✅ Should have some form of support for varying material properties (e.g. fetching them from a buffer)
    - 📖 Varying material/object properties can be supplied through buffers.
- ⚠️ Mirror/Portal rendering
  - 🛑 Ideally should be as efficient as possible for VR - use single pass rather than two separate renders
  - ⚠️ More flexible on render method
    - ✅ Skewed matrix with render to texture (current method)
      - 📖 Custom camera/projection and render target control are available.
    - ✅ Scissor/Stencil is a potential alternative method
      - 📖 Scissor and stencil state are exposed by the graphics API.

**Ideal:**
- ⚠️ HDR display output
  - 📖 Color-space and HDR-oriented output paths are present in examples, including Rec.2020-style display output handling. Platform coverage needs integration validation.
- 🛑 Multiple Window support
  - 📖 Window and swapchain abstractions are present, but a demonstrated multi-window rendering workflow was not found.
- 🛑 Mesh shaders (with meshlets) pipeline
  - 📖 Meshlet data structures exist, but a working mesh shader pipeline was not found.
  - ⚠️ Provide fallback for older GPU's
    - 📖 Visibility-buffer and meshlet-like CPU/GPU culling paths could act as a practical fallback, but this is not a mesh shader fallback system.
- ✅ Better alpha sorting/blending handling
  - 📖 The examples include order-independent transparency work, including weighted blended OIT/AOIT-related paths.

**Nice to have:**
- ✅ Forms of static / dynamic occlusion culling
  - 📖 Visibility-buffer examples include GPU triangle filtering/culling, and the graphics API exposes occlusion query support.
- ⚠️ Ability to re-render the same mesh multiple times with different materials efficiently
  - ⚠️ Perform frustum / occlusion culling just once
    - 📖 This is plausible with visibility-buffer/indirect rendering, but a general reusable material-stacking system was not found.
  - 🛑 For skinned meshes - transform vertices just once
    - 📖 GPU skinning examples exist, but a built-in system for sharing skinned vertices across repeated material passes was not found.
- ✅ Reversed floating point depth buffer
  - 📖 Reverse-Z projection and greater-or-equal depth testing patterns are present in the camera and rendering examples.

## Shader pipeline
**Required:**
- ⚠️ Shader pipeline must be isolated enough (or made to be isolated) so it can be invoked from our own code at runtime to dynamically compile shaders
  - 📖 The resource loader and FSL tooling expose shader loading/compilation paths, cached bytecode loading, and build-time shader translation. Runtime integration appears possible, but a stable engine-facing runtime shader compiler API was not found.
- ✅ Compute Shader support
  - ✅ Needs to support wave intrinsics
    - 📖 A dedicated WaveIntrinsics unit test and wave-ops tooling flags are present.

**Ideal:**
- 🛑 Slang support
- ✅ SPIR-V
- ✅ Existing PBR/PBS shaders
  - 📖 The Material Playground and related shaders cover PBR-style material workflows and image-based lighting.

## Post processing
To match feature parity, the rendering pipeline needs to support the same/similar post processing filters as our current render currently does.

**Required:**
- 🛑 Bloom
  - 🛑 Must support working with HDR values (above 1.0)
- ⚠️ Motion Blur
  - ✅ Motion vector based
    - 📖 Motion vector generation and temporal resources are present in screen-space reflection and denoising examples.
  - 🛑 Support camera blur
  - 🛑 Support object blur
  - 🛑 Support skinned mesh blur
  - 🛑 Ideally supported both in VR & desktop, but desktop is sufficient
- ✅ Ambient Occlusion
  - 📖 Ambient occlusion is present in visibility-buffer examples.
- ✅ Anti-aliasing
  - ✅ MSAA (given by the rendering path)
    - 📖 MSAA sample counts and programmable MSAA examples are present.
- ⚠️ VR & single pass support
  - 📖 VR multiview support is present, but each post-processing effect would need validation or adaptation for multiview rendering.

**Nice to have:**
- ✅ Screen space reflections
- 🛑 Some form of screen space realtime GI
  - 📖 Public screen-space realtime GI support was not found. Advanced RTX GI middleware is described as non-public.

**Not required:**
- ✅ Anti-aliasing (these are largely needed for deferred rendering)
- 🛑 TAA
- 🛑 FXAA
- 🛑 CTAA
- 🛑 SMAA
- 🛑 …

## Rendering components / entities
Note that the structure does not need to match the current ones, but the engine needs to have API's and structures so we can replicate functionality of these. It is *not* required that the render specifically has those entities implemented, but rather that it has features that allow them to be implemented.

### Mesh Rendering
**Must have:**
- ✅ Triangle topology
- ✅ Point topology
- ⚠️ Some form of submesh support
  - ⚠️ Allow specifying materials for each submesh - the whole mesh is rendered & culled as a unit, but each submesh is its own
    - 📖 Geometry draw-argument arrays and app-managed material IDs support this pattern, but a high-level submesh/material component was not found.
- ⚠️ Shadow rendering support (depending on material support)
  - ✅ Single sided
  - ✅ Dual sided
  - ✅ None
  - ⚠️ Shadow only
    - 📖 Shadow-only behavior can be implemented by drawing only into shadow passes, but a dedicated material flag was not found.

### Skinned Mesh Rendering
These requirements are on top of standard mesh rendering.

**Required:**
- ✅ 1-4 bone support
- 🛑 Blendshape support
  - 🛑 Multi-frame support (each blendshape goes through several frames)
  - 🛑 Positions, Normals & Tangents
- ✅ GPU accelerated for good performance
  - 📖 The skinning unit test uses GPU skinning with joint and weight vertex attributes.

**Nice to have:**
- 🛑 More than 4 bones
- 🛑 Blendshape support
  - 🛑 UV’s, Colors
  - 🛑 Ability to cache rarely changing blendshapes to avoid recomputations
  - 🛑 Ability to only compute affected vertices & skip rest
- 🛑 Dual Quaternion Skinning #487

### Lights
**Required:**
- Supported types:
  - ✅ Point
  - 🛑 Spot
    - 📖 Spot light support was not found in the public examples.
  - ✅ Directional
- Supported features:
  - ⚠️ Hard & Soft realtime shadows for for all types
    - 📖 Directional shadow examples and multiple shadow filtering approaches are present. Realtime shadows for every light type were not found.
  - ⚠️ Multiple instances of each type
    - 📖 Multiple point lights are present in clustered lighting examples. Multiple shadowed directional/spot light systems were not found.
  - 🛑 Light cookies for point & spot lights
- ✅ Control over lighting falloff
  - 📖 Light shaders expose attenuation/falloff-style calculations through shader code.

**Nice to have:**
- 🛑 RGB light cookies
- ⚠️ Baked shadowmaps
  - 📖 The low-level texture/render-target API can use precomputed shadow textures, but a baked shadowmap pipeline was not found.
- ⚠️ Control over realtime shadowmap rendering
  - ⚠️ Ideally the pipeline could render point/spot light shadowmaps once each frame and share them for each camera/view that’s rendered that frame
    - 📖 Pass and render-target control are available, but a reusable point/spot shadowmap sharing system was not found.
- 🛑 Realtime area/polygonal lights

### Cameras
**Required:**
- ✅ Support rendering additional views (other than primary view) into a render texture
- ✅ Double buffering support (cameras see the contents of their own render texture from previous frame)
  - 📖 The API supports app-managed render targets and previous-frame textures; temporal examples use this pattern.
- ⚠️ Support camera stacking with render order (depth)
  - ✅ Multiple cameras must be able to render into the same texture
  - ⚠️ Order must be able to be defined (e.g. with depth value)
    - 📖 Render order is controlled by command submission order. A camera-depth stacking abstraction was not found.
  - ✅ Must support viewport configuration (rendering to a sub-section of the render texture)
  - ✅ Must support different clear methods (none, depth only, color/skybox)

### Reflection probes
**Required:**
- ⚠️ Reflection probe that affects specular lighting on materials (notably PBS materials)
  - 📖 PBR material examples use environment/specular cubemap lighting, but a reflection probe component was not found.
- ✅ Varying smoothness support stored in mip maps
- 🛑 Real time reflection probe rendering
  - 🛑 Support for time slicing when rendering (doesn't need to be baked in as long as we have control where individual phases of render occur)
- ✅ Baked reflection probes
  - ✅ Using cubemap with mipmaps storing different smoothness levels
- ⚠️ Rendering reflection probe to cubemap asset
  - 📖 Cubemap render targets/textures are supported, but a ready-made reflection-probe capture system was not found.
- 🛑 Control over when is reflection probe rendered

## Scene model API
Since FrooxEngine has its own scene model, the ideal state for the renderer is to be minimalistic and hold only data absolutely necessary for the rendering itself. If needed, we can have our own code to hold this data on top of the renderer if it doesn’t naturally hold it between frames.

**Required:**
- ⚠️ Mechanism to filter which entities are rendered for each camera
  - 📖 This is primarily application-managed through submitted buffers, draw lists, visibility-buffer passes, and per-view rendering. A high-level renderer-owned entity filter API was not found.

**Ideal:**
- ✅ Flat scene description with no transform hierarchy - we submit 4x4 matrices for each entity to be rendered
  - 📖 Examples generally submit app-managed transforms, matrices, and buffers rather than relying on a renderer-owned transform hierarchy.
- ✅ Submitted poses are in world space and can be shared between multiple cameras/views
- ⚠️ Renderer performs camera frustum culling & sorting
  - 📖 Visibility-buffer examples perform GPU-driven triangle/object filtering, but a general scene-wide culling and sorting service was not found.

**Possible alternatives:**
- ✅ Camera frustum culling & sorting is performed on engine side & fully sorted list of “render commands / draw calls” is submitted to the renderer for each camera / view

**Not viable:**
- Individual drawcalls and render commands are proxied from the main process
  - This would occur too much overhead for IPC
  - It’s possible that the renderer works this way on its side and we write code to hold the simplified scene state, but the scene state needs to be submitted in bulk

## Asset/resource support

### Textures
**Required:**
- Texture types
  - ✅ 2D
  - ✅ Cubemap
  - ✅ 3D
- ✅ API to upload arbitrary texture data to the GPU provided in a byte buffer
  - ✅ Support for uploading texture sub-regions
  - ✅ Support for specifying max mip level to currently use
  - ✅ Support for dynamic updates (textures can be updated anytime - important for procedural textures)
- ✅ Support for common compressed formats (depending on HW)
  - ✅ Uncompressed formats (e.g. RGBA32/ARGB32)
  - ✅ Block compressed formats (BCx, ETCx, ASTC...)

**Ideal:**
- ✅ Uploads can be fully done from background threads - engine takes care of any necessary synchronization
  - 📖 The resource loader supports asynchronous texture, buffer, and geometry loading with synchronization tokens.

### Meshes
**Required:**
- Supported vertex attributes
  - ✅ Positions
  - ✅ Normals
  - ✅ Tangents
  - ✅ UV’s
    - ✅ At least 8 channels
    - ✅ 2-4 dimensions per each channel
  - ✅ Colors
- Supported topologies
  - ✅ Triangles
  - ✅ Points
- ⚠️ Skinned mesh data
  - ✅ Support binding 1-4 bone transforms to each vertex
  - 🛑 Support blendshape data in some form
    - 🛑 Positions, normals, tangents
    - 🛑 Support more than one frame for blendshape (progression of blendshape goes through multiple frames)

### Video Textures
Since Resonite supports video playback and this is handled by the renderer (due to GPU texture resources being updated) so this support will need to be handled by the new renderer as well.

This will most likely have to be implemented - e.g. through doing integration with libVLC.

**Required:**
- 🛑 Video texture support integrated with the rendering pipeline
  - 🛑 Ability to control the playback (playing, looping, playback position)
  - 🛑 Ability to select audio track that’s decoded
- 🛑 Ability to get raw audio data
- 🛑 Support for both local file playback & streaming
  - 🛑 Streaming video files from a web endpoint
  - 🛑 Supporting live video streams (like rtmp)

**Ideal:**
- 🛑 Ideally libVLC integration for maximum compatibility
  - Can be potentially other playback engines, but the format support needs to be similar
- 🛑 HW/GPU decoding support
- 🛑 Specifying separate video & audio stream URL’s

## Input handling
Since the renderer will be providing the user a window and interfacing with some of the input devices (e.g. VR), we need to be able to proxy various inputs.

If these are not supported, they should be trivial to implement in Phase 3.

**Required:**
- ⚠️ Keyboard support (including Unicode)
  - ✅ Individual key press events
  - ✅ Type delta
  - 🛑 IME composition
- ⚠️ Mouse support
  - ✅ At least 5 mouse buttons
  - ⚠️ Scroll wheel (including horizontal ones)
    - 📖 Scroll wheel support is present; horizontal scroll support was not confirmed.
  - ✅ Absolute position
  - ✅ Relative delta
- 🛑 VR input support
  - 🛑 Access state of all controller buttons/elements
  - 🛑 Haptics support
  - 🛑 Access to full skeleton of the hand (for supported controllers)
- ⚠️ Touch support
  - ✅ Multi-touch
  - 🛑 Ideally have access to other properties like pressure

**Nice to have:**
- ✅ Gamepad support
- 🛑 Pen/Stylus support
  - 🛑 Pressure, tilt etc...

## Other "nice to have"
- 🛑 Spout support
- 🛑 Video encoding support (e.g. FFmpeg/NVENC integration and so on)
