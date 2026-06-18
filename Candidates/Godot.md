# Godot
**Repository:** https://github.com/godotengine/godot

**License:** MIT

## Basic Overview
**Programming language(s):**
- ✅ C++
  - 📖 Primary engine, renderer, platform, and editor implementation.
- ✅ C#
  - 📖 Supported through the Mono/.NET module, but not the renderer core.
- ⚠️ Objective-C++, Java, Python, SCons, and platform glue
  - 📖 Auxiliary platform/build/editor code rather than the renderer language.

**Shading language(s):**
- ✅ Godot shader language
  - 📖 Primary high-level material/shader authoring language.
- ✅ Internal GLSL-like renderer shaders
  - 📖 Used heavily by the RenderingDevice renderers under `servers/rendering/renderer_rd`.
- ⚠️ Native GLSL shader files
  - 📖 Importable as `RDShaderFile` for low-level `RenderingDevice` use, not as normal `.gdshader` materials.
- ✅ SPIR-V
  - 📖 `RenderingDevice` exposes SPIR-V shader compilation and shader creation APIs.
- 🛑 Slang

**Rendering paths:**
- ✅ Forward+ / clustered forward
- ✅ Mobile forward
- ⚠️ Compatibility / OpenGL
  - 📖 Useful as a Godot fallback path, but OpenGL is not acceptable as the primary target for these requirements.

**Graphics API's:**
- ✅ Vulkan
- ✅ Direct3D 12
- ✅ Metal
- ⚠️ OpenGL 3 / OpenGL ES 3 / ANGLE
  - 📖 Present through the Compatibility renderer, but not a good fit for the required renderer target.

**Supported platforms:**
- ✅ Windows
- ✅ Linux / BSD
- ✅ macOS
- ✅ Android
- ✅ iOS
- ✅ Web
- ✅ visionOS

**Oldest supported hardware:**
- ❓ Exact minimum GPU generation
- ⚠️ RenderingDevice renderers require modern Vulkan, Direct3D 12, or Metal-capable hardware
  - 📖 The Compatibility path targets older OpenGL 3.3 / OpenGL ES 3.0 class hardware, but that path does not satisfy the no-OpenGL target preference.

**VR API's:**
- ✅ OpenXR
- ✅ WebXR
- ✅ MobileVR

**Activity:**
- Rough contributor count: ✅ 3,727 git authors
- Rough user/community size: 🗨 Large community

## AI Use
How AI is actually used in the project, whether AI contributions are allowed, and whether the renderer is meaningfully human-designed or substantially AI-coded without technical oversight. Note that post-2021 public git history cannot reliably prove that merged code was never AI-assisted. Godot should not be described as AI-free.

- Project policy
  - Godot includes AI disclosure language in `CONTRIBUTING.md` and the pull request template. Autonomous AI agents are expected to disclose themselves, and PR AI use should include a description of how it was used.
- Observed evidence
  - Git history includes at least one 2025 merged PR branch with a `claude/...` branch name (`a627ee6c10`). This is a weak provenance indication by itself, but it supports treating Godot as AI-aware rather than AI-free.
- Evaluation impact
  - Godot is a mature, heavily reviewed engine with explicit AI disclosure controls. The evaluation should focus on maintainer review, renderer quality, and disclosure policy rather than assuming no AI-assisted contributions exist.

# Existing usage / projects made with this
Any high profile (or low profile if they're particularly relevant) projects and companies using this project. Include links and any notes that are important.

# General notes
Anything noteworthy that's not related to the any of the requirements directly should be added to this section.

## Positive highlights
- Mature full engine with a large community, permissive MIT license, and broad platform support.
- Strong renderer feature coverage: Forward+ clustered rendering, low-level `RenderingDevice`, GI options, reflection probes, post-processing, texture/mesh resource APIs, and XR support.
- OpenXR integration is substantial, including action maps, haptics, hand tracking, foveation/VRS-related code paths, and multiple graphics API bindings.
- Input and windowing coverage is broad across desktop, mobile, web, and XR.

## Potential concerns
- Godot is a full game engine, not a renderer-only library. Separating the renderer from Godot's scene tree, resource system, servers, and main loop would likely be a major integration effort.
- Some features are available only through low-level `RenderingDevice` rather than the high-level material/scene APIs.
- Compatibility/OpenGL support is useful for Godot users but should not count toward the preferred target renderer path.
- Performance parity with the current Unity renderer needs content-specific benchmarks.

## Other notes
- Godot's lower-level `RenderingServer` and `RenderingDevice` APIs are the most relevant pieces for renderer-backend integration.


# New Renderer Requirements
This section compares Godot against the renderer requirements.

> [!IMPORTANT]
> Please read the [README.md](/README.md) on how to contribute and work with this document!

> [!NOTE]
> This is based on the [REQUIREMENTS.md](/REQUIREMENTS.md) document, but with some notes & context removed to keep it cleaner.
> Check the matching features in that document for more context / information!

## VR Rendering
**Required:**
- ✅ Allow implementation of dynamic VR/desktop switching
  - 📖 `Viewport.use_xr` and XR interface selection allow XR output to be enabled/disabled at the viewport level.

**Ideal:**
- ✅ Single-pass stereo rendering
  - 📖 `RenderingDevice` framebuffer formats support `view_count >= 2` multiview for VR when the backend supports it.
- ✅ Canted displays rendering support (e.g. Pimax)
  - 📖 XR interfaces expose per-view projections, so asymmetric/canted-eye projections are representable.
- ⚠️ Foveated rendering support
  - 📖 XR VRS helpers and OpenXR foveation-related extensions exist, but availability depends on backend, hardware, and XR runtime support, using extensions known supported by SteamVR and Meta Quest (Mobile only).

## Render pipeline
**Required:**
- 🗨 General performance on par or better with current Unity renderer
  - 📖 Godot has a production renderer, but parity must be measured with Resonite/FrooxEngine content and target hardware.
- ✅ Some level of control over graphics pipeline rendering
  - 📖 `RenderingDevice`, `RenderingServer`, and compositor effects expose significant low-level and render-pass control, though the built-in scene renderer remains engine-managed.
- ⚠️ Ability to control the order of rendering and sorting
  - 📖 Godot exposes render priorities, transparency sorting behavior, layers, and compositor hooks, but exact Resonite-style render grouping would likely need custom renderer integration.
  - ⚠️ **Ideal:** Ability to create a "group/batch" of render entities that are rendered at once as an unit and have their own internal sorting order

- ⚠️ Stencil buffer support
  - 📖 `RenderingDevice` exposes full depth/stencil pipeline state, dynamic stencil masks/references, and stencil draw flags. High-level `BaseMaterial3D` stencil controls are more limited.
  - Supported operations must match the currently exposed ones through materials
    - ✅ 8-bit integer (0 to 255)
      - 📖 Implemented as 8-bit stencil formats such as S8 and depth/stencil formats.
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
- ⚠️ LOD support
  - 📖 `ArrayMesh` supports surface LOD dictionaries, and `GeometryInstance3D` supports visibility ranges and fade modes.
  - ⚠️ Per-render switching between render entities depending on relative size on the screen
  - ⚠️ Ideally blending support (e.g. through material/shader properties)
- ✅ Some form of Global Illumination (GI) support
  - 📖 Includes VoxelGI, LightmapGI, SDFGI, SSIL, reflection probes, and environment lighting depending on renderer and scene setup.
- ⚠️ GPU instancing
  - 📖 `MultiMesh` efficiently renders many instances of the same mesh/material with per-instance transform, color, and custom data. It is explicit, not a fully automatic render-entity batching model.
  - ⚠️ Ideally fully automated from the render entities
  - ✅ Efficiently render large number of entities with the same material & mesh
  - ✅ Should have some form of support for varying material properties (e.g. fetching them from a buffer)
- ⚠️ Mirror/Portal rendering
  - 📖 SubViewports, render textures, custom camera projections, and stencil/scissor-capable low-level APIs exist, but Godot does not expose a dedicated portal/mirror system.
  - 🗨 Ideally should be as efficient as possible for VR - use single pass rather than two separate renders
  - ⚠️ More flexible on render method
    - ✅ Skewed matrix with render to texture (current method)
    - ⚠️ Scissor/Stencil is a potential alternative method

**Ideal:**
- ✅ HDR display output
  - 📖 `DisplayServer` exposes HDR output support on Linux Wayland, macOS, iOS, visionOS, and Windows when the platform/display supports it.
- ✅ Multiple Window support
  - 📖 `DisplayServer` lists multiple-window support for Windows, macOS, and Linux X11.
- 🛑 Mesh shaders (with meshlets) pipeline
  - 📖 No implemented mesh shader rendering pipeline found.
  - 🛑 Provide fallback for older GPU's
- ⚠️ Better alpha sorting/blending handling
  - 📖 Godot has transparency sorting, render priority, and alpha/depth modes, but no advanced order-independent transparency path is exposed.

**Nice to have:**
- ✅ Forms of static / dynamic occlusion culling
  - 📖 Viewports can enable 3D occlusion culling with occluder instances.
- ⚠️ Ability to re-render the same mesh multiple times with different materials efficiently
  - 📖 Multiple surfaces/materials and next-pass materials exist, but no direct path found for culling once and rendering the same mesh with several materials.
  - ❓ Perform frustum / occlusion culling just once
  - ❓ For skinned meshes - transform vertices just once
- ✅ Reversed floating point depth buffer
  - 📖 Reverse-Z projection/depth correction support is present in the renderer math code.

## Shader pipeline
**Required:**
- ✅ Shader pipeline must be isolated enough (or made to be isolated) so it can be invoked from our own code at runtime to dynamically compile shaders
  - 📖 `RenderingDevice` exposes runtime shader compilation/creation APIs. Godot's full material shader pipeline is still tied into the engine.
- ✅ Compute Shader support
  - 📖 `RenderingDevice` exposes compute pipelines and compute dispatch lists.
  - ⚠️ Needs to support wave intrinsics
    - 📖 Backend/internal shader code exposes subgroup/wave capability paths, but user-facing/runtime support is backend- and shader-path-dependent.

**Ideal:**
- 🛑 Slang support
- ✅ SPIR-V
- ✅ Existing PBR/PBS shaders
  - 📖 Godot's 3D material system includes standard PBR materials and renderer shaders.

## Post processing
To match feature parity, the rendering pipeline needs to support the same/similar post processing filters as our current render currently does.

**Required:**
- ✅ Bloom
  - ✅ Must support working with HDR values (above 1.0)
    - 📖 Godot's glow/bloom and tonemapping paths operate in the HDR scene pipeline.
- ⚠️ Motion Blur
  - 📖 Motion vectors are generated for TAA/FSR2-style temporal effects, but Godot does not expose a built-in motion blur post-process.
  - ⚠️ Motion vector based
  - 🛑 Support camera blur
  - 🛑 Support object blur
  - 🛑 Support skinned mesh blur
  - 🛑 Ideally supported both in VR & desktop, but desktop is sufficient
- ✅ Ambient Occlusion
  - 📖 SSAO is exposed through environment settings.
- ✅ Anti-aliasing
  - ✅ MSAA (given by the rendering path)
- ⚠️ VR & single pass support
  - 📖 XR multiview exists, but each post-process needs validation in the XR/multiview path.

**Nice to have:**
- ✅ Screen space reflections
  - 📖 SSR is supported in Forward+.
- ✅ Some form of screen space realtime GI
  - 📖 SSIL and SDFGI are present in the Forward+ renderer.

**Not required:**
- ✅ Anti-aliasing (these are largely needed for deferred rendering)
  - ✅ TAA
  - ✅ FXAA
  - 🛑 CTAA
  - ✅ SMAA
  - …

## Rendering components / entities
Note that the structure does not need to match the current ones, but the engine needs to have API's and structures so we can replicate functionality of these. It is *not* required that the render specifically has those entities implemented, but rather that it has features that allow them to be implemented.

### Mesh Rendering
**Must have:**
- ✅ Triangle topology
- ✅ Point topology
- ✅ Some form of submesh support
  - 📖 Godot meshes are split into surfaces, each with its own primitive type and material.
  - ✅ Allow specifying materials for each submesh - the whole mesh is rendered & culled as a unit, but each submesh is its own
- ✅ Shadow rendering support (depending on material support)
  - ✅ Single sided
  - ✅ Dual sided
  - ✅ None
  - ✅ Shadow only
  - 📖 `GeometryInstance3D` exposes off/on/double-sided/shadows-only shadow casting settings.

### Skinned Mesh Rendering
These requirements are on top of standard mesh rendering.

**Required:**
- ✅ 1-4 bone support
- ✅ Blendshape support
  - 🛑 Multi-frame support (each blendshape goes through several frames)
  - ✅ Positions, Normals & Tangents
- ✅ GPU accelerated for good performance
  - 📖 The RenderingServer and renderer storage have mesh instances, skeleton RIDs, blend shape weights, and RD skeleton shader paths.

**Nice to have:**
- ✅ More than 4 bones
  - 📖 Mesh/SurfaceTool support 8 bone weights with `ARRAY_FLAG_USE_8_BONE_WEIGHTS`.
- ⚠️ Blendshape support
  - 🛑 UV’s, Colors
  - ❓ Ability to cache rarely changing blendshapes to avoid recomputations
  - ❓ Ability to only compute affected vertices & skip rest
- 🛑 Dual Quaternion Skinning #487

### Lights
**Required:**
- Supported types:
  - ✅ Point
  - ✅ Spot
  - ✅ Directional
- Supported features:
  - ✅ Hard & Soft realtime shadows for all types
    - 📖 Light shadow settings include realtime shadows, shadow blur, PCSS-style size controls, and directional split modes.
  - ✅ Multiple instances of each type
  - ✅ Light cookies for point & spot lights
    - 📖 `Light3D.light_projector` supports cookie/gobo-style projection in Forward+ and Mobile when shadows are enabled.
- ✅ Control over lighting falloff
  - 📖 Omni, spot, and area lights expose attenuation controls.

**Nice to have:**
- ✅ RGB light cookies
  - 📖 Light projectors use `Texture2D`.
- ⚠️ Baked shadowmaps
  - 📖 LightmapGI supports baked lighting and experimental directional shadowmasking, but this is not a general baked shadowmap system for every light type.
- ⚠️ Control over realtime shadowmap rendering
  - 📖 Shadow sizes, split modes, masks, bias, fade, and per-light enablement exist, but exact per-frame sharing across multiple cameras/views would need renderer-level validation.
  - ❓ Ideally the pipeline could render point/spot light shadowmaps once each frame and share them for each camera/view that’s rendered that frame
- ⚠️ Realtime area/polygonal lights
  - 📖 `AreaLight3D` exists with texture and PCSS-style soft shadow support, but Mobile and Compatibility have limitations.

### Cameras
**Required:**
- ✅ Support rendering additional views (other than primary view) into a render texture
  - 📖 SubViewports/ViewportTextures provide offscreen rendering.
- ⚠️ Double buffering support (cameras see the contents of their own render texture from previous frame)
  - 📖 Viewport textures exist, but explicit previous-frame feedback would likely require manual ping-ponging.
- ⚠️ Support camera stacking with render order (depth)
  - 📖 Godot has Viewports/SubViewports and active cameras, but no direct Unity-style camera stack.
  - ⚠️ Multiple cameras must be able to render into the same texture
  - ⚠️ Order must be able to be defined (e.g. with depth value)
  - ✅ Must support viewport configuration (rendering to a sub-section of the render texture)
  - ✅ Must support different clear methods (none, depth only, color/skybox)

### Reflection probes
**Required:**
- ✅ Reflection probe that affects specular lighting on materials (notably PBS materials)
- ✅ Varying smoothness support stored in mip maps
  - 📖 Reflection probes store blurred cubemap versions for different roughness levels.
- ✅ Real time reflection probe rendering
  - ✅ Support for time slicing when rendering (doesn't need to be baked in as long as we have control where individual phases of render occur)
    - 📖 `UPDATE_ONCE` generates the radiance map over six frames; `UPDATE_ALWAYS` updates every frame.
- ✅ Baked reflection probes
  - ✅ Using cubemap with mipmaps storing different smoothness levels
- ⚠️ Rendering reflection probe to cubemap asset
  - 📖 ReflectionProbe internally captures cubemaps, but no direct high-level path found to save a probe as a cubemap asset.
- ⚠️ Control over when is reflection probe rendered
  - 📖 Probes support `UPDATE_ONCE` and `UPDATE_ALWAYS`; forcing a one-shot refresh is indirect.

## Scene model API
Since FrooxEngine has its own scene model, the ideal state for the renderer is to be minimalistic and hold only data absolutely necessary for the rendering itself. If needed, we can have our own code to hold this data on top of the renderer if it doesn’t naturally hold it between frames.

**Required:**
- ⚠️ Mechanism to filter which entities are rendered for each camera
  - 📖 Cameras and reflection probes use cull masks against `VisualInstance3D` layers. Cull masks are limited in number.

**Ideal:**
- ⚠️ Flat scene description with no transform hierarchy - we submit 4x4 matrices for each entity to be rendered
  - 📖 `RenderingServer` works with RIDs and instance transforms, but Godot's standard scene model is a hierarchical scene tree.
- ⚠️ Submitted poses are in world space and can be shared between multiple cameras/views
  - 📖 Feasible at the server level, but not the default high-level Node3D workflow.
- ✅ Renderer performs camera frustum culling & sorting

**Possible alternatives:**
 - ⚠️ Camera frustum culling & sorting is performed on engine side & fully sorted list of “render commands / draw calls” is submitted to the renderer for each camera / view
   - 📖 This is possible only if using `RenderingDevice` directly or building a custom renderer layer; the built-in scene renderer does not expose a simple bulk draw-command submission interface for this use case.

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
  - ⚠️ Support for uploading texture sub-regions
    - 📖 Texture copy APIs support regions; direct CPU upload is primarily whole texture/layer/mipmap update.
  - ✅ Support for specifying max mip level to currently use
  - ✅ Support for dynamic updates (textures can be updated anytime - important for procedural textures)
- ✅ Support for common compressed formats (depending on HW)
  - ✅ Uncompressed formats (e.g. RGBA32/ARGB32)
  - ✅ Block compressed formats (BCx, ETCx, ASTC...)

**Ideal:**
- ⚠️ Uploads can be fully done from background threads - engine takes care of any necessary synchronization
  - 📖 Separate `RenderingDevice` instances can be created for threaded work, but integration with the main renderer upload path would need design/validation.

### Meshes
**Required:**
- Supported vertex attributes
  - ✅ Positions
  - ✅ Normals
  - ✅ Tangents
  - ⚠️ UV’s
    - 🛑 At least 8 channels
      - 📖 Godot exposes UV and UV2 plus four custom vertex channels, not eight dedicated UV channels.
    - ⚠️ 2-4 dimensions per each channel
      - 📖 UV/UV2 are Vector2; custom channels can carry 1-4 component data depending on format.
  - ✅ Colors
- Supported topologies
  - ✅ Triangles
  - ✅ Points
- ✅ Skinned mesh data
  - ✅ Support binding 1-4 bone transforms to each vertex
  - ✅ Support blendshape data in some form
    - ✅ Positions, normals, tangents
    - 🛑 Support more than one frame for blendshape (progression of blendshape goes through multiple frames)

### Video Textures
Since Resonite supports video playback and this is handled by the renderer (due to GPU texture resources being updated) so this support will need to be handled by the new renderer as well.

This will most likely have to be implemented - e.g. through doing integration with libVLC.

**Required:**
- ⚠️ Video texture support integrated with the rendering pipeline
  - 📖 Built-in support is Ogg Theora (`.ogv`) plus formats supplied by GDExtension plugins.
  - ✅ Ability to control the playback (playing, looping, playback position)
  - ✅ Ability to select audio track that’s decoded
- ⚠️ Ability to get raw audio data
  - 📖 `VideoStreamPlayback` exposes audio mixing hooks, but a Resonite-style raw audio extraction API would likely need custom integration.
- ⚠️ Support for both local file playback & streaming
  - ✅ Local file playback
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
- ✅ Keyboard support (including Unicode)
  - ✅ Individual key press events
  - ✅ Type delta
  - ✅ IME composition
- ✅ Mouse support
  - ✅ At least 5 mouse buttons
  - ✅ Scroll wheel (including horizontal ones)
  - ✅ Absolute position
  - ✅ Relative delta
- ✅ VR input support
  - ✅ Access state of all controller buttons/elements
  - ✅ Haptics support
  - ✅ Access to full skeleton of the hand (for supported controllers)
- ✅ Touch support
  - ✅ Multi-touch
  - ✅ Ideally have access to other properties like pressure

**Nice to have:**
- ✅ Gamepad support
- ✅ Pen/Stylus support
  - ✅ Pressure, tilt etc...

## Other "nice to have"
- 🛑 Spout support
- ⚠️ Video encoding support (e.g. FFmpeg/NVENC integration and so on)
  - 📖 Godot has MovieWriter infrastructure, PNG/WAV frame output, and an OGV/Theora writer, but no FFmpeg/NVENC integration.
