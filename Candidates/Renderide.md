# Renderide
**Repository:** https://github.com/DoubleStyx/Renderide

**License:** MIT

## Basic Overview
**Programming language(s):**
- ✅ Rust
  - 📖 Primary renderer implementation.
- ✅ C#
  - 📖 Shared IPC type generation only.

**Shading language(s):**
- ✅ WGSL
  - 📖 Primary shader language.
- 🗨 Naga / Naga Oil
  - 📖 Used for WGSL composition, reflection, and validation in the renderer build pipeline.
- 🛑 Direct GLSL shader authoring
  - 📖 Can be added but is not currently exposed/planned.
- 🛑 Direct SPIR-V shader authoring
  - 📖 Can be added but is not currently exposed/planned.
- 🛑 Slang

**Rendering paths:**
- ✅ Clustered forward

**Graphics API's:**
- ✅ wgpu
  - ✅ Vulkan
  - ✅ Metal
  - ✅ DirectX 12
  - ✅ OpenGL

**Supported platforms:**
- ✅ Windows
- ✅ MacOS
  - 📖 Native support.
- ✅ Linux
  - 📖 Native support.
- 🔜 iOS / Android
  - 📖 Mobile would generally require single-process support from the engine.

**Oldest supported hardware:**
- 🗨 Tested with GTX 1080 Ti and newer
  - 📖 Theoretically should work on GTX 600 or newer

**VR API's:**
- ✅ OpenXR

**Activity:**
- Rough contributor count: ✅ 15 human contributors / 16 total local git authors
- Rough user/community size: 52 stars

## AI Use
How AI is actually used in the project, whether AI contributions are allowed, and whether the renderer is human-designed or vibe-coded without technical oversight.

- The README AI Policy states that Renderide does not accept AI-generated or AI-assisted source code, shaders, documentation, tests, issues, pull requests, or review comments.

# Existing usage / projects made with this
Any high profile (or low profile if they're particularly relevant) projects and companies using this project. Include links and any notes that are important.

- N/A - designed directly for interfacing with FrooxEngine and Resonite content requirements.

# General notes
Anything noteworthy that's not related to the any of the requirements directly should be added to this section.

## Positive highlights
- Tailor-made for Resonite compatibility.
- OpenXR-first architecture with stereo multiview and head-tracked input in the core path.
- Data-driven render graph and material pipeline.
- Tracy CPU/GPU profiling feature is built in behind an opt-in Cargo feature.

## Potential concerns
- Needs additional testing on a broad range of content to verify correctness.

## Other notes
- Engineered around existing content compatibility and needs a mechanism for evaluating currently engine-unsupported features like light probes. This could be prototyped using the CI software runners via the `renderide-test` crate.


# New Renderer Requirements
This is the "meat" of the evaluation - going through our list of requirements and checking how well the renderer matches the required features.

> [!IMPORTANT]
> Please read the [README.md](/README.md) on how to contribute and work with this document!

> [!NOTE]
> This is based on the [REQUIREMENTS.md](/REQUIREMENTS.md) document, but with some notes & context removed to keep it cleaner.
> When evaluating features, check the matching features in that document for more context / information!

## VR Rendering
**Required:**
- ✅ Allow implementation of dynamic VR/desktop switching

**Ideal:**
- ✅ Single-pass stereo rendering
  - 📖 OpenXR uses a two-layer swapchain and multiview render paths.
- ✅ Canted displays rendering support (e.g. Pimax)
  - 📖 OpenXR per-eye asymmetric FOV tangents are used to build reverse-Z projection matrices.
- 🔜 Foveated rendering support
  - 📖 Requires an upstream PR to wgpu to add variable rate shading support. Alternatives include quad view rendering.

## Render pipeline
**Required:**
- 🗨 General performance on par or better with current Unity renderer
  - 📖 Generally better but varies on hardware and content. More optimizations are being added over time.
- ✅ Some level of control over graphics pipeline rendering
- ✅ Ability to control the order of rendering and sorting
  - ✅ **Ideal:** Ability to create a "group/batch" of render entities that are rendered at once as an unit and have their own internal sorting order

- ✅ Stencil buffer support
  - Supported operations must match the currently exposed ones through materials
    - ✅ 8 byte integer (0 to 255)
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
- 🔜 LOD support
  - 🔜 Per-render switching between render entities depending on relative size on the screen
  - c Ideally blending support (e.g. through material/shader properties)
- ✅ Some form of Global Illumination (GI) support
  - 📖 Specular IBL/reflection-probe lighting is present.
  - 📖 Ambient SH2 is supported with an optional experimental feature for reflection probe SH2 blending.
- ✅ GPU instancing
  - ✅ Ideally fully automated from the render entities
  - ✅ Efficiently render large number of entities with the same material & mesh
  - ✅ Should have some form of support for varying material properties (e.g. fetching them from a buffer)
- 🔜 Mirror/Portal rendering
  - 📖 Offscreen render-texture cameras are implemented, but dedicated mirror behavior is in progress.
  - 🔜 Ideally should be as efficient as possible for VR - use single pass rather than two separate renders
  - 🔜 More flexible on render method
    - 🔜 Skewed matrix with render to texture (current method)
    - 🔜 Scissor/Stencil is a potential alternative method

**Ideal:**
- 🔜 HDR display output
  - 📖 HDR scene color and tonemapping exist, but HDR swapchain/display output is in progress.
- 🔜 Multiple Window support
- 🔜 Mesh shaders (with meshlets) pipeline
  - 🔜 Provide fallback for older GPU's
- ✅ Better alpha sorting/blending handling

**Nice to have:**
- ✅ Forms of static / dynamic occlusion culling
  - 📖 Hi-Z occlusion and frustum culling are present.
- ✅ Ability to re-render the same mesh multiple times with different materials efficiently
  - ✅ Perform frustum / occlusion culling just once
  - ✅ For skinned meshes - transform vertices just once
- ✅ Reversed floating point depth buffer

## Shader pipeline
**Required:**
- ✅ Shader pipeline must be isolated enough (or made to be isolated) so it can be invoked from our own code at runtime to dynamically compile shaders
  - 📖 WGSL material reflection, permutation, and pipeline caching are isolated in the material system, but this is currently for embedded/composed WGSL rather than an exposed arbitrary runtime shader API.
  - 📖 An arbitrary shader API is in progress.
- ✅ Compute Shader support
  - 🔜 Needs to support wave intrinsics

**Ideal:**
- 🛑 Slang support
  - 📖 Slang does not support `MULTIVIEW` for wgsl yet; wgsl already cross-compiles to different backends
    - 📖 The MULTIVIEW keyword could possibly be injected as a post-step in the emission, but this could be more complex.
- 🛑 SPIR-V
  - 📖 Can be added but is not currently exposed/planned.
- ✅ Existing PBR/PBS shaders

## Post processing
To match feature parity, the rendering pipeline needs to support the same/similar post processing filters as our current render currently does.

**Required:**
- ✅ Bloom
  - ✅ Must support working with HDR values (above 1.0)
- 🔜 Motion Blur
  - ✅ Motion vector based
  - ✅ Support camera blur
  - 🔜 Support object blur
  - 🔜 Support skinned mesh blur
  - ✅ Ideally supported both in VR & desktop, but desktop is sufficient
    - 📖 The shader has mono and multiview variants, and the config exposes a VR allow toggle.
- ✅ Ambient Occlusion
  - 📖 GTAO is implemented.
- ✅ Anti-aliasing
  - ✅ MSAA (given by the rendering path)
- ✅ VR & single pass support

**Nice to have:**
- 🔜 Screen space reflections
- 🔜 Some form of screen space realtime GI

**Not required:**
  - 🔜 Anti-aliasing (these are largely needed for deferred rendering)
  - 🔜 TAA
  - 🔜 FXAA
  - 🔜 CTAA
  - 🔜 SMAA
  - …

## Rendering components / entities
Note that the structure does not need to match the current ones, but the engine needs to have API's and structures so we can replicate functionality of these. It is *not* required that the render specifically has those entities implemented, but rather that it has features that allow them to be implemented.

### Mesh Rendering
**Must have:**
- ✅ Triangle topology
- ✅ Point topology
- ✅ Some form of submesh support
  - ✅ Allow specifying materials for each submesh - the whole mesh is rendered & culled as a unit, but each submesh is its own
- 🔜 Shadow rendering support (depending on material support)
  - 🔜 Single sided
  - 🔜 Dual sided
  - 🔜 None
  - 🔜 Shadow only
    - 📖 Shadow-only renderers are represented/skipped for color draws, but shadow-map rendering passes are in progress.

### Skinned Mesh Rendering
These requirements are on top of standard mesh rendering.

**Required:**
- ✅ 1-4 bone support
- ✅ Blendshape support
  - ✅ Multi-frame support (each blendshape goes through several frames)
  - ✅ Positions, Normals & Tangents
- ✅ GPU accelerated for good performance
  - 📖 Implemented with compute passes for skinning/blendshape deform.

**Nice to have:**
- 🔜 More than 4 bones
- ✅ Blendshape support
  - 🔜 UV's, Colors
  - ✅ Ability to cache rarely changing blendshapes to avoid recomputations
  - ✅ Ability to only compute affected vertices & skip rest
- 🔜 Dual Quaternion Skinning #487

### Lights
**Required:**
- Supported types:
  - ✅ Point
  - ✅ Spot
  - ✅ Directional
- Supported features:
  - 🔜 Hard & Soft realtime shadows for for all types
  - ✅ Multiple instances of each type
  - 🔜 Light cookies for point & spot lights
- 🔜 Control over lighting falloff
  - 📖 Unity-style point/spot attenuation is implemented, but arbitrary falloff control is in progress.

**Nice to have:**
- 🔜 RGB light cookies
- 🔜 Baked shadowmaps
- 🔜 Control over realtime shadowmap rendering
  - 🔜 Ideally the pipeline could render point/spot light shadowmaps once each frame and share them for each camera/view that’s rendered that frame
- 🔜 Realtime area/polygonal lights

### Cameras
**Required:**
- ✅ Support rendering additional views (other than primary view) into a render texture
- ⚠️ Double buffering support (cameras see the contents of their own render texture from previous frame)
  - 📖 The render-texture path avoids same-pass self-sampling, but explicit camera self-history behavior is in progress as a full double-buffer feature.
- ✅ Support camera stacking with render order (depth)
  - ✅ Multiple cameras must be able to render into the same texture
  - ✅ Order must be able to be defined (e.g. with depth value)
  - ✅ Must support viewport configuration (rendering to a sub-section of the render texture)
  - ✅ Must support different clear methods (none, depth only, color/skybox)

### Reflection probes
**Required:**
- ✅ Reflection probe that affects specular lighting on materials (notably PBS materials)
- ✅ Varying smoothness support stored in mip maps
- ✅ Real time reflection probe rendering
  - ✅ Support for time slicing when rendering (doesn't need to be baked in as long as we have control where individual phases of render occur)
- ✅ Baked reflection probes
  - ✅ Using cubemap with mipmaps storing different smoothness levels
- ✅ Rendering reflection probe to cubemap asset
- ✅ Control over when is reflection probe rendered

## Scene model API
Since FrooxEngine has its own scene model, the ideal state for the renderer is to be minimalistic and hold only data absolutely necessary for the rendering itself. If needed, we can have our own code to hold this data on top of the renderer if it doesn’t naturally hold it between frames.

**Required:**
- ✅ Mechanism to filter which entities are rendered for each camera

**Ideal:**
- 🗨 Flat scene description with no transform hierarchy - we submit 4x4 matrices for each entity to be rendered
  - 📖 Renderide holds scene-layer transform/render-space state, but renderer-side draw planning uses submitted transform data.
- 🗨 Submitted poses are in world space and can be shared between multiple cameras/views
- ✅ Renderer performs camera frustum culling & sorting

**Possible alternatives:**
- ⚠️ Camera frustum culling & sorting is performed on engine side & fully sorted list of "render commands / draw calls" is submitted to the renderer for each camera / view

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
- Supported topologies
  - ✅ Triangles
  - ✅ Points
- ✅ Skinned mesh data
  - ✅ Support binding 1-4 bone transforms to each vertex
  - ✅ Support blendshape data in some form
    - ✅ Positions, normals, tangents
    - ✅ Support more than one frame for blendshape (progression of blendshape goes through multiple frames)

### Video Textures
Since Resonite supports video playback and this is handled by the renderer (due to GPU texture resources being updated) so this support will need to be handled by the new renderer as well.

This will most likely have to be implemented - e.g. through doing integration with libVLC.

**Required:**
- ✅ Video texture support integrated with the rendering pipeline
  - 📖 Implemented behind the opt-in `video-textures` feature using GStreamer. Stock builds allocate a black GPU placeholder without decoding.
  - ✅ Ability to control the playback (playing, looping, playback position)
  - ✅ Ability to select audio track that’s decoded
- ✅ Ability to get raw audio data
- ✅ Support for both local file playback & streaming
  - ✅ Streaming video files from a web endpoint
  - ✅ Supporting live video streams (like rtmp)

**Ideal:**
- 🛑 Ideally libVLC integration for maximum compatibility
  - 📖 The video implementation uses GStreamer, not libVLC.
- 🔜 HW/GPU decoding support
  - 📖 Currently uses a CPU-copy sink on all platforms.
- 🔜 Specifying separate video & audio stream URL's

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
  - 🔜 Access to full skeleton of the hand (for supported controllers)
    - 📖 Hand IPC types and synthesized hands exist, but full controller skeleton tracking is in progress.
- 🔜 Touch support
  - 🔜 Multi-touch
  - 🔜 Ideally have access to other properties like pressure

**Nice to have:**
- 🔜 Gamepad support
- 🔜 Pen/Stylus support
  - 🔜 Pressure, tilt etc...

## Other "nice to have"
- 🔜 Spout support
- 🔜 Video encoding support (e.g. FFmpeg/NVENC integration and so on)
