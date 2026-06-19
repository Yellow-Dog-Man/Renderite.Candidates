# Stride
**Repository:** https://github.com/stride3d/stride

**License:** MIT

## Basic Overview
**Programming language(s):** Primarily C#, with native/platform glue where needed.

**Shading language(s):** SDSL, Stride's shader language, with HLSL-style effects. The repo also has HLSL-to-SPIR-V/GLSL tooling for Vulkan paths.

**Rendering paths:** Forward renderer. The Level 10 default compositor uses clustered forward rendering for point and spot lights, with simpler Level 9 fallback paths.

**Graphics API's:** Direct3D 11, Direct3D 12, Vulkan, and Null.

**Supported platforms:** Windows, Linux, macOS, Android, iOS, and UWP are represented in the SDK/platform files. VR device backends are currently tied to Direct3D 11 paths.

**Oldest supported hardware:** Graphics profiles range from Level 9.1 through Level 11.2. DX11-class hardware is the realistic target for the fuller renderer; Level 9 support exists but is much more limited.

**VR API's:** OpenXR, OpenVR, Oculus OVR, Windows Mixed Reality, and a dummy device backend.

**Activity:**
- Rough contributor count: ~150
- Rough user/community size: Moderate

## AI Use
How AI is actually used in the project, whether AI contributions are allowed, and whether the renderer is meaningfully human-designed or substantially AI-coded without technical oversight. Note that post-2021 public git history cannot reliably prove that merged code was never AI-assisted. Stride has explicit AI/tooling evidence and should not be described as AI-free.

- Project policy
  - No direct ban on AI-generated contributions was found. Instead, the repo includes guidance and tooling for AI-assisted workflows.
- Observed evidence
  - The repo contains `.github/copilot-instructions.md`, Claude/Anthropic screenshot-comparison fallback code and CI, a source comment noting a test header parser was generated with ChatGPT, Claude co-author trailers in 2026 commits, and history that tracked then removed `CLAUDE.md` / `.claude/`.
- Evaluation impact
  - Stride is a mature maintainer-reviewed engine, but AI-assisted tooling is visibly part of recent project workflow. It as reviewed/maintainer-owned code with AI provenance uncertainty, not strictly an AI-free renderer.

# Existing usage / projects made with this
The README points to community resources, demos, articles, shaders, physics examples, and the Stride Community Toolkit.

# General notes
Anything noteworthy that's not related to the any of the requirements directly should be added to this section.

## Positive highlights
- Mostly in C# - same that Resonite is written in.
- MIT licensed and under the .NET Foundation.
- Mature engine architecture with Game Studio, an asset pipeline, render stages, render features, graphics compositor assets, shader compiler services, and broad platform/input/media modules.
- The graphics compositor is flexible enough to route cameras, render textures, clear passes, post effects, and custom render features without changing the whole renderer.
- Stencil support, compute shaders, PBR materials, clustered forward lighting, light probes, render-to-texture, video textures, and broad input support are already in the repo.

## Potential concerns
- The active graphics APIs are Direct3D 11, Direct3D 12, and Vulkan. There is no Metal backend, and Direct3D 11 is still the Windows default in the SDK files.
- OpenXR/OpenVR/Oculus/WMR support exists, but the runtime device initialization paths are currently Direct3D 11-gated.
- No foveated rendering/VRS path, mesh shader path, Slang integration, blendshape rendering, dedicated motion blur effect, dedicated reflection probe component, or built-in occlusion culling path was found in the repo.
- Stride is a full game engine, not a small renderer library. A Resonite integration would likely need to adapt or bypass pieces of the ECS, asset, and content systems.

## Other notes
- The repo includes `Stride.Core.Shaders` for shader parsing/type analysis/conversion and a Vulkan shader path that uses SPIR-V.


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
  - 📖 The VR renderer can copy or skip the desktop mirror, and it uses separate VR render views. A built-in per-entity VR-vs-desktop visibility workflow was not found beyond normal render masks/stages.

**Ideal:**
- ⚠️ Single-pass stereo rendering
  - 📖 Stride shares culling/lighting work between the two eyes in the VR path, but the forward renderer still draws each eye separately.
- ⚠️ Canted displays rendering support (e.g. Pimax)
  - 📖 OpenXR/OpenVR/Oculus paths read per-eye projections from the runtime, so asymmetric/canted projection data can flow through. It is still rendered as separate eye draws rather than an efficient single-pass path.
- 🛑 Foveated rendering support
  - 📖 No VRS/foveation renderer path was found in the repo.

## Render pipeline
**Required:**
- 🗨 General performance on par or better with current Unity renderer
  - 📖 Stride has useful performance indicators: clustered forward lighting, render stages/features, instancing support, command lists, and GPU tests. Actual parity with Resonite's Unity renderer would need benchmarking on Resonite-like scenes.
- ✅ Some level of control over graphics pipeline rendering
  - 📖 Graphics compositors, render stages, render features, scene renderers, and custom effects provide good control over where rendering work happens.
- ⚠️ Ability to control the order of rendering and sorting
  - 📖 Render stages and sort modes support front-to-back, back-to-front, and state-change sorting. A direct built-in equivalent to Resonite's full render queue plus SortingOrder model was not found.
  - ⚠️ **Ideal:** Ability to create a "group/batch" of render entities that are rendered at once as a unit and have their own internal sorting order
    - 📖 This looks implementable with custom render stages/features, but a built-in grouped render unit with its own internal sort was not found in the repo.

- ✅ Stencil buffer support
  - Supported operations must match the currently exposed ones through materials
    - ✅ 8-bit integer (0 to 255)
      - 📖 The API uses `byte` stencil masks/values and depth-stencil formats with 8-bit stencil.
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
  - 🛑 Per-render switching between render entities depending on relative size on the screen
  - 🛑 Ideally blending support (e.g. through material/shader properties)
  - 📖 Model comments mention LOD, but active screen-size based LOD switching or LOD blending was not found in the render repo.
- ✅ Some form of Global Illumination (GI) support
  - 📖 Light probes and skybox environment lighting exist. Light probe runtime data can be generated and coefficients can be updated at runtime.
- ⚠️ GPU instancing
  - ⚠️ Ideally fully automated from the render entities
  - ✅ Efficiently render large number of entities with the same material & mesh
  - ⚠️ Should have some form of support for varying material properties (e.g. fetching them from a buffer)
  - 📖 Stride has instancing processors/features and calls instanced draw commands. Automatic batching of arbitrary matching render entities and arbitrary per-instance material properties would need integration work.
- ⚠️ Mirror/Portal rendering
  - 🛑 Ideally should be as efficient as possible for VR - use single pass rather than two separate renders
  - ⚠️ More flexible on render method
    - ⚠️ Skewed matrix with render to texture (current method)
    - ⚠️ Scissor/Stencil is a potential alternative method
  - 📖 Render-to-texture scene renderers, camera renderers, viewports, stencil, and custom compositor passes are available. A dedicated mirror/portal renderer was not found in the repo.

**Ideal:**
- ⚠️ HDR display output
  - 📖 Internal HDR render targets are used for post effects, and the Direct3D presenter has HDR output color-space support. The presenter docs say HDR output is currently Direct3D-only.
- ⚠️ Multiple Window support
  - 📖 The graphics layer has presenter and swapchain abstractions, but the game platform centers on one main window. A polished multi-window workflow was not found in the repo.
- 🛑 Mesh shaders (with meshlets) pipeline
  - 🛑 Provide fallback for older GPU's
  - 📖 Mesh shader support is listed only as a future graphics profile comment, not as an implemented pipeline.
- 🛑 Better alpha sorting/blending handling
  - 📖 Standard transparent back-to-front sorting exists. OIT, weighted blended transparency, or a similar alpha sorting workaround was not found in the repo.

**Nice to have:**
- 🛑 Forms of static / dynamic occlusion culling
  - 📖 Frustum culling exists through `VisibilityGroup`, but static/dynamic occlusion culling was not found in the repo.
- ⚠️ Ability to re-render the same mesh multiple times with different materials efficiently
  - ⚠️ Perform frustum / occlusion culling just once
  - ⚠️ For skinned meshes - transform vertices just once
  - 📖 Multiple material passes and reused mesh draw data exist, but the repo comments still note that skinning could be shared between meshes in a model. This does not look like full material-stacking efficiency today.
- ⚠️ Reversed floating point depth buffer
  - 📖 Inverse depth states exist, but a global reversed-Z pipeline was not found in the repo.

## Shader pipeline
**Required:**
- ✅ Shader pipeline must be isolated enough (or made to be isolated) so it can be invoked from our own code at runtime to dynamically compile shaders
  - 📖 The engine has shader compiler services, SDSL/effect assets, dynamic effect instances, and shader parser/type analysis/conversion code.
- ⚠️ Compute Shader support
  - 🛑 Needs to support wave intrinsics
  - 📖 Compute shaders are supported and tested. Wave/subgroup intrinsics were not found in the repo.

**Ideal:**
- 🛑 Slang support
- ✅ SPIR-V
  - 📖 Vulkan shader paths use SPIR-V, with HLSL-to-SPIR-V build tooling.
- ✅ Existing PBR/PBS shaders
  - 📖 The material system includes metalness/specular workflows, microfacet specular models, environment lighting, and related PBS/PBR material features.

## Post processing
To match feature parity, the rendering pipeline needs to support the same/similar post processing filters as our current render currently does.

**Required:**
- ✅ Bloom
  - ✅ Must support working with HDR values (above 1.0)
  - 📖 The forward renderer uses floating-point render targets when post effects are enabled, and bloom is part of the post-processing stack.
- 🛑 Motion Blur
  - 🛑 Motion vector based
  - 🛑 Support camera blur
  - 🛑 Support object blur
  - 🛑 Support skinned mesh blur
  - 🛑 Ideally supported both in VR & desktop, but desktop is sufficient
  - 📖 Velocity buffer support exists for temporal anti-aliasing, but no motion blur effect path was found in the repo.
- ✅ Ambient Occlusion
- ✅ Anti-aliasing
  - ✅ MSAA (given by the rendering path)
- ⚠️ VR & single pass support
  - 📖 VR post effects can run per eye through the VR path, but single-pass stereo itself is not implemented.

**Nice to have:**
- ✅ Screen space reflections
  - 📖 Implemented as screen-space local reflections.
- 🛑 Some form of screen space realtime GI

**Not required:**
  - ✅ Anti-aliasing (these are largely needed for deferred rendering)
  - ✅ TAA
  - ✅ FXAA
  - 🛑 CTAA
  - 🛑 SMAA
  - ...

## Rendering components / entities
Note that the structure does not need to match the current ones, but the engine needs to have API's and structures so we can replicate functionality of these. It is *not* required that the render specifically has those entities implemented, but rather that it has features that allow them to be implemented.

### Mesh Rendering
**Must have:**
- ✅ Triangle topology
- ✅ Point topology
- ⚠️ Some form of submesh support
  - ⚠️ Allow specifying materials for each submesh - the whole mesh is rendered & culled as a unit, but each submesh is its own
  - 📖 Models contain meshes with material indices. The renderer registers render meshes separately, so the exact "whole mesh culled as a unit, submesh rendered separately" behavior would need care.
- ⚠️ Shadow rendering support (depending on material support)
  - ✅ Single sided
  - ✅ Dual sided
  - ✅ None
  - ⚠️ Shadow only
  - 📖 Rasterizer culling and shadow caster flags exist. Shadow-only looks possible through custom stages/material setup, but a simple built-in shadow-only render mode was not found.

### Skinned Mesh Rendering
These requirements are on top of standard mesh rendering.

**Required:**
- ✅ 1-4 bone support
- 🛑 Blendshape support
  - 🛑 Multi-frame support (each blendshape goes through several frames)
  - 🛑 Positions, Normals & Tangents
- ✅ GPU accelerated for good performance
  - 📖 Skinning uses blend indices/weights and shader skinning for position/normal/tangent paths.

**Nice to have:**
- ⚠️ More than 4 bones
  - 📖 More than 4 bones per mesh/skeleton is supported through bone arrays. More than 4 bone weights per vertex was not found.
- 🛑 Blendshape support
  - 🛑 UV's, Colors
  - 🛑 Ability to cache rarely changing blendshapes to avoid recomputations
  - 🛑 Ability to only compute affected vertices & skip rest
- 🛑 Dual Quaternion Skinning #487

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
    - 📖 Spot light projective textures are supported. Point light cookies were not found in the repo.
- ⚠️ Control over lighting falloff
  - 📖 Built-in point/spot attenuation exists, and custom shader work should be possible. A user-configurable Unity-style legacy falloff curve was not found.

**Nice to have:**
- ⚠️ RGB light cookies
  - 📖 Spot projective textures can carry color. Point light cookies were not found.
- 🛑 Baked shadowmaps
- ⚠️ Control over realtime shadowmap rendering
  - ⚠️ Ideally the pipeline could render point/spot light shadowmaps once each frame and share them for each camera/view that is rendered that frame
  - 📖 Shadow map renderer hooks and per-view shadow collection exist. Full Resonite-style scheduling/sharing would need verification and likely integration work.
- 🛑 Realtime area/polygonal lights

### Cameras
**Required:**
- ✅ Support rendering additional views (other than primary view) into a render texture
  - 📖 `RenderTextureSceneRenderer`, camera slots, and external camera renderers support this pattern.
- ⚠️ Double buffering support (cameras see the contents of their own render texture from previous frame)
  - 📖 This should be implementable with two render textures, but a built-in camera double-buffer mode was not found.
- ⚠️ Support camera stacking with render order (depth)
  - ⚠️ Multiple cameras must be able to render into the same texture
  - ⚠️ Order must be able to be defined (e.g. with depth value)
  - ✅ Must support viewport configuration (rendering to a sub-section of the render texture)
  - ⚠️ Must support different clear methods (none, depth only, color/skybox)
  - 📖 Compositor ordering, viewports, render textures, and clear renderers exist. A Unity-like camera stack with depth ordering was not found as a built-in workflow.

### Reflection probes
**Required:**
- 🛑 Reflection probe that affects specular lighting on materials (notably PBS materials)
- ⚠️ Varying smoothness support stored in mip maps
- ⚠️ Real time reflection probe rendering
  - ⚠️ Support for time slicing when rendering (doesn't need to be baked in as long as we have control where individual phases of render occur)
- ⚠️ Baked reflection probes
  - ⚠️ Using cubemap with mipmaps storing different smoothness levels
- ⚠️ Rendering reflection probe to cubemap asset
- ⚠️ Control over when is reflection probe rendered
  - 📖 Cubemap rendering, skybox environment lighting, and GGX specular prefiltering exist. A dedicated reflection probe component/workflow affecting PBS materials was not found in the repo.

## Scene model API
Since FrooxEngine has its own scene model, the ideal state for the renderer is to be minimalistic and hold only data absolutely necessary for the rendering itself. If needed, we can have our own code to hold this data on top of the renderer if it doesn't naturally hold it between frames.

**Required:**
- ✅ Mechanism to filter which entities are rendered for each camera
  - 📖 `RenderView.CullingMask`/render masks provide per-camera filtering.

**Ideal:**
- ⚠️ Flat scene description with no transform hierarchy - we submit 4x4 matrices for each entity to be rendered
  - 📖 Stride's engine-side model is ECS with transform hierarchy. The renderer ultimately consumes world matrices/render objects, but the public engine model is not flat.
- ✅ Submitted poses are in world space and can be shared between multiple cameras/views
  - 📖 Render objects carry world matrices, and the VR path shares culling from a common view before copying visibility to eye views.
- ✅ Renderer performs camera frustum culling & sorting

**Possible alternatives:**
 - ⚠️ Camera frustum culling & sorting is performed on engine side & fully sorted list of "render commands / draw calls" is submitted to the renderer for each camera / view
   - 📖 Stride internally builds render object visibility and sorted render nodes. Exposing this as a clean external command submission API would need adapter work.

**Not viable:**
 - Individual drawcalls and render commands are proxied from the main process
  - This would occur too much overhead for IPC
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
  - 📖 Texture creation, texture views, mip-level views, `SetData`, and `UpdateSubResource` with `ResourceRegion` exist. A high-level progressive texture streaming/max-active-mip workflow would need integration work.
- ✅ Support for common compressed formats (depending on HW)
  - ✅ Uncompressed formats (e.g. RGBA32/ARGB32)
  - ✅ Block compressed formats (BCx, ETCx, ASTC...)

**Ideal:**
- ⚠️ Uploads can be fully done from background threads - engine takes care of any necessary synchronization
  - 📖 The graphics feature set exposes multi-threading/command-list related capabilities, but a clear fully background-threaded upload workflow was not found.

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
  - 📖 Vertex elements are semantic/index/format based, so these streams can be represented. Material shader conventions may still need custom stream wiring for uncommon layouts.
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
- ⚠️ Video texture support integrated with the rendering pipeline
  - ✅ Ability to control the playback (playing, looping, playback position)
  - 🛑 Ability to select audio track that's decoded
  - 📖 FFmpeg/MediaCodec video texture support exists, with play/pause/stop/seek/loop/range controls and texture updates into render targets.
- 🛑 Ability to get raw audio data
  - 📖 Audio stream metadata exists in the FFmpeg layer, but the video instance does not expose decoded raw audio buffers for an external audio system.
- ⚠️ Support for both local file playback & streaming
  - ⚠️ Streaming video files from a web endpoint
  - ⚠️ Supporting live video streams (like rtmp)
  - 📖 FFmpeg can open URLs, but Stride's asset path expects video files from the virtual file system. Full web/live-stream playback parity would need more work.

**Ideal:**
- 🛑 Ideally libVLC integration for maximum compatibility
  - 📖 FFmpeg and Android MediaCodec support exist instead.
- ⚠️ HW/GPU decoding support
  - 📖 Hardware decoding paths exist in the FFmpeg video texture code, but exact platform coverage would need testing.
- 🛑 Specifying separate video & audio stream URL's

## Input handling
Since the renderer will be providing the user a window and interfacing with some of the input devices (e.g. VR), we need to be able to proxy various inputs.

If these are not supported, they should be trivial to implement in Phase 3.

**Required:**
- ✅ Keyboard support (including Unicode)
  - ✅ Individual key press events
  - ✅ Type delta
  - ✅ IME composition
  - 📖 Text input events include normal input and composition data.
- ⚠️ Mouse support
  - ✅ At least 5 mouse buttons
  - ⚠️ Scroll wheel (including horizontal ones)
  - ✅ Absolute position
  - ✅ Relative delta
  - 📖 SDL mouse support handles five buttons, absolute movement, relative movement, and vertical wheel. Horizontal wheel handling was not found in the repo.
- ⚠️ VR input support
  - ✅ Access state of all controller buttons/elements
  - ⚠️ Haptics support
  - 🛑 Access to full skeleton of the hand (for supported controllers)
  - 📖 Touch controller abstractions expose buttons, axes, poses, trigger/grip, and haptics. OpenVR/Oculus haptics are implemented; OpenXR/WMR haptics are marked as none. OpenVR generated bindings include skeletal APIs, but Stride's controller abstraction does not expose full hand skeletons.
- ⚠️ Touch support
  - ✅ Multi-touch
  - ⚠️ Ideally have access to other properties like pressure
  - 📖 Pointer/touch events exist. Pressure support was not found in the repo.

**Nice to have:**
- ✅ Gamepad support
- ⚠️ Pen/Stylus support
  - ⚠️ Pressure, tilt etc...
  - 📖 UWP pointer code recognizes pen input, but pressure/tilt support was not found.

## Other "nice to have"
- 🛑 Spout support
- 🛑 Video encoding support (e.g. FFmpeg/NVENC integration and so on)
  - 📖 The repo contains image encoding and video decoding paths, but no renderer-facing video encoding pipeline was found.
