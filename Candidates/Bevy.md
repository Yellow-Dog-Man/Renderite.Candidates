# Bevy
**Repository:** https://github.com/bevyengine/bevy

**License:** MIT, Apache 2.0

## Basic Overview
**Programming language(s):** Rust

**Shading language(s):** WGSL primarily. Feature-gated GLSL, SPIR-V, and WESL shader formats are also present.

**Rendering paths:** Clustered forward / Forward+ PBR by default, optional deferred opaque path and prepasses, experimental meshlet visibility-buffer renderer.

**Graphics API's:** Vulkan, DX12, and Metal through wgpu. WebGPU/WebGL2 for web builds; GLES/native GL is available as an optional fallback path, but is not Bevy's preferred native renderer path.

**Supported platforms:** Windows, Linux, macOS, Web, iOS, Android. Android support is present, but Quest/OpenXR support is not built into Bevy.

**Oldest supported hardware:** Roughly WebGPU/wgpu-capable hardware. Standard rendering can run on mainstream Vulkan/DX12/Metal GPUs and WebGL2 fallback targets; experimental meshlets require newer GPU features.

**VR API's:** 🛑 None built in.

**Activity:**
- Rough contributor count: ~1,659 local git authors in the checked-out repository.
- Rough user/community size: ~46.1k GitHub stars and ~4.6k forks as of 2026-05-19; official Discord, Reddit, GitHub Discussions, Bevy Assets, and Bevy Foundation/supporter ecosystem exist.

## AI Use
Bevy is a human-designed open source engine with an explicit AI policy. The policy rejects AI-generated code and non-code assets for Bevy Organization repositories, except trivial autocomplete-style suggestions that are indistinguishable from ordinary IDE assistance.

Source: https://bevy.org/learn/contribute/policies/ai/

# Existing usage / projects made with this
Bevy has a sizeable community ecosystem rather than a short list of obvious AAA/high-profile shipped products. The official Bevy Assets catalog lists community games, apps, tooling, plugins, editors, and renderer extensions.

Source: https://bevy.org/assets/

# General notes
Anything noteworthy that's not related to the any of the requirements directly should be added to this section.

## Positive highlights
- Sizeable contributor community - unlikely to disappear soon and actively developed
- Modern, Rust based
- Modular ECS/render-graph architecture with strong low-level renderer extension points
- Permissive MIT/Apache-2.0 licensing
- Strong stance against unreviewed AI-generated contributions

## Potential concerns
- Bevy's own README warns that it is still early-stage, with missing features, sparse docs in places, and breaking releases approximately every 3 months.
- Bevy is a full ECS game engine, not a minimal flat rendering library. Integrating it as a renderer-only backend would require careful ownership boundaries.
- Built-in XR, video playback, Spout, and video encoding support are missing.
- Several renderer capabilities exist only through low-level hooks, feature flags, experimental plugins, or custom render graph work.

## Other notes


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
  - 📖 Bevy has per-camera render targets, render layers, custom projections, external `TextureView` render targets, and camera ordering. This is enough for a custom XR integration, but no built-in VR runtime exists.

**Ideal:**
- 🛑 Single-pass stereo rendering
  - 📖 Local render pass descriptors use `multiview_mask: None`, and no built-in stereo/multiview path was found.
- ⚠️ Canted displays rendering support (e.g. Pimax)
  - 📖 Custom camera projections can represent canted/off-axis frusta, but there is no built-in XR display integration or efficient stereo path.
- 🛑 Foveated rendering support
  - 📖 No built-in foveated rendering or VRS integration was found locally or in official Bevy sources.

## Render pipeline
**Required:**
- 🗨 General performance on par or better with current Unity renderer
  - 📖 Bevy has many performance-oriented systems, including batching, clustered lighting, GPU preprocessing, occlusion culling, and experimental meshlets, but no apples-to-apples Resonite/Unity benchmark was found.
- ✅ Some level of control over graphics pipeline rendering
  - 📖 Bevy exposes render graphs, render phases, custom render graph nodes, custom render phases, specialized pipelines, custom materials, and compute/render pipeline descriptors.
- ✅ Ability to control the order of rendering and sorting
  - 📖 Cameras have render order, render phases support binned and sorted phases, and custom phase items can provide custom queue/sort/draw behavior.
		- ⚠️ **Ideal:** Ability to create a “group/batch” of render entities that are rendered at once as an unit and have their own internal sorting order
		  - 📖 Bevy has batch sets/bins and multidraw support internally, but no direct high-level Render Queue group object matching this requirement.

- ⚠️ Stencil buffer support
	- 📖 Underlying wgpu pipeline state supports the requested stencil compare/ops/masks, and Bevy exposes `DepthStencilState` through pipeline descriptors. StandardMaterial does not expose a high-level stencil material API.
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
- ⚠️ LOD support
	- ⚠️ Per-render switching between render entities depending on relative size on the screen
	  - 📖 Bevy has `VisibilityRange` HLOD/dither crossfade based on distance to the camera, not screen-relative size.
	- ✅ Ideally blending support (e.g. through material/shader properties)
	  - 📖 Visibility ranges include a dithered crossfade margin.
- ⚠️ Some form of Global Illumination (GI) support
  - 📖 Bevy supports lightmaps, irradiance volumes, reflection/environment maps, and experimental Solari raytraced GI. Full runtime-baking parity would still need custom work.
- ⚠️ GPU instancing
	- ✅ Ideally fully automated from the render entities
	- ✅ Efficiently render large number of entities with the same material & mesh
	- ⚠️ Should have some form of support for varying material properties (e.g. fetching them from a buffer)
	  - 📖 Bevy batches compatible mesh/material entities automatically. Arbitrary per-instance material data is possible with custom shaders/pipelines, but not a direct StandardMaterial feature-parity API.
- ⚠️ Mirror/Portal rendering
	- 🛑 Ideally should be as efficient as possible for VR - use single pass rather than two separate renders
	- ⚠️ More flexible on render method
		- ✅ Skewed matrix with render to texture (current method)
		- ⚠️ Scissor/Stencil is a potential alternative method
		  - 📖 Render-to-texture, custom projection, camera culling inversion, and a mirror example exist. Stencil/scissor-style portal behavior would require custom pipeline/render phase work.

**Ideal:**
- 🛑 HDR display output
  - 📖 Bevy supports HDR intermediate render textures, but its `Hdr` docs explicitly say HDR display output is not currently supported.
- ✅ Multiple Window support
- ⚠️ Mesh shaders (with meshlets) pipeline
  - ✅ Provide fallback for older GPU's
  - 📖 Bevy has an experimental meshlet renderer with a standard renderer fallback. It requires recent GPU features, currently works only on Vulkan/Metal, and is not a general hardware mesh-shader pipeline.
- ⚠️ Better alpha sorting/blending handling
  - 📖 Bevy has alpha modes and order-independent transparency, but OIT has limitations such as no MSAA compatibility in the example path.

**Nice to have:**
- ✅ Forms of static / dynamic occlusion culling
  - 📖 Bevy has occlusion culling support and meshlet-level occlusion in the experimental meshlet renderer.
- 🛑 Ability to re-render the same mesh multiple times with different materials efficiently
        - 🛑 Perform frustum / occlusion culling just once
        - 🛑 For skinned meshes - transform vertices just once
  - 📖 This can be emulated by submitting the same mesh multiple times, but no efficient material-stacking path with shared culling/skinning was found.
- ✅ Reversed floating point depth buffer
  - 📖 Bevy's 3D camera/depth pipeline uses reversed-Z semantics with a `Depth32Float` depth buffer.

## Shader pipeline
**Required:**
- ✅ Shader pipeline must be isolated enough (or made to be isolated) so it can be invoked from our own code at runtime to dynamically compile shaders
  - 📖 Bevy has shader assets, `Shader::from_wgsl`, `from_glsl`, `from_spirv`, pipeline descriptors, shader defs, and a pipeline cache.
- ⚠️ Compute Shader support
	- ⚠️ Needs to support wave intrinsics
	  - 📖 Compute shaders are supported on native/WebGPU-capable backends, but not WebGL2. Bevy/wgpu expose subgroup support through GPU features and Bevy uses subgroup/quad operations internally when available, so wave/subgroup availability is backend/hardware dependent.

**Ideal:**
- 🛑 Slang support
  - 📖 No built-in Slang support was found locally or in official docs/search results.
- ⚠️ SPIR-V
  - 📖 Feature-gated SPIR-V loading/passthrough exists, but passthrough is Vulkan-only and more limited than WGSL in Bevy's shader asset pipeline.
- ✅ Existing PBR/PBS shaders

## Post processing
To match feature parity, the rendering pipeline needs to support the same/similar post processing filters as our current render currently does.

**Required:**
- ✅ Bloom
	- ✅ Must support working with HDR values (above 1.0)
- ⚠️ Motion Blur
	- ✅ Motion vector based
	- ✅ Support camera blur
	- ✅ Support object blur
	- ✅ Support skinned mesh blur
	- ⚠️ Ideally supported both in VR & desktop, but desktop is sufficient
	  - 📖 Motion-vector prepass and motion blur exist for desktop-style rendering. VR/single-pass support is missing because Bevy has no built-in XR/single-pass renderer.
- ✅ Ambient Occlusion
- ✅ Anti-aliasing
	- ✅ MSAA (given by the rendering path)
- 🛑 VR & single pass support
  - 📖 No built-in single-pass stereo renderer was found.

**Nice to have:**
- ✅ Screen space reflections
- 🔜 Some form of screen space realtime GI
  - 📖 Bevy has experimental Solari raytraced GI, but this is not a mature screen-space GI parity feature.

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
- ⚠️ Some form of submesh support
	- ⚠️ Allow specifying materials for each submesh - the whole mesh is rendered & culled as a unit, but each submesh is its own
	  - 📖 Bevy's `Mesh` maps closely to a glTF primitive. glTF multi-primitive meshes are represented as multiple mesh entities/primitives, so material-per-primitive is supported but not one mesh culled as a single submesh unit by default.
- ⚠️ Shadow rendering support (depending on material support)
	- ✅ Single sided
	- ✅ Dual sided
	- ✅ None
	- ⚠️ Shadow only
	  - 📖 `NotShadowCaster`, `NotShadowReceiver`, cull modes, and double-sided materials exist. Shadow-only rendering would require custom material/render-phase work.

### Skinned Mesh Rendering
These requirements are on top of standard mesh rendering.

**Required:**
- ✅ 1-4 bone support
- ⚠️ Blendshape support
	- ⚠️ Multi-frame support (each blendshape goes through several frames)
	  - 📖 Bevy supports multiple morph targets and animated weights, but no explicit named multi-frame blendshape concept was found.
	- ✅ Positions, Normals & Tangents
- ✅ GPU accelerated for good performance

**Nice to have:**
- 🛑 More than 4 bones
  - 📖 Bevy supports many joints in a skeleton, but only 4 joint indices/weights per vertex.
- 🛑 Blendshape support
	- 🛑 UV’s, Colors
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
	- ⚠️ Hard & Soft realtime shadows for for all types
	  - 📖 Hard shadows are supported. Soft shadows are supported through PCSS-related features/settings but are experimental/noisy/feature-gated in current Bevy.
	- ✅ Multiple instances of each type
	- ✅ Light cookies for point & spot lights
	  - 📖 Light textures are feature-gated with `pbr_light_textures`.
- ⚠️ Control over lighting falloff
  - 📖 Lights expose range/radius/intensity and spot inner/outer angles. Matching Unity/legacy arbitrary falloff would require custom shader/light model work.

**Nice to have:**
- ✅ RGB light cookies
- ⚠️ Baked shadowmaps
  - 📖 Bevy has lightmaps, but no direct baked-shadowmap entity/API equivalent was found.
- ⚠️ Control over realtime shadowmap rendering
	- ⚠️ Ideally the pipeline could render point/spot light shadowmaps once each frame and share them for each camera/view that’s rendered that frame
	  - 📖 Bevy has shared point/spot shadow passes, while directional/cascaded shadows are per-view. Fine-grained scheduling modes would need renderer work.
- ⚠️ Realtime area/polygonal lights
  - 📖 Bevy has `RectLight`; arbitrary polygonal area lights were not found.

### Cameras
**Required:**
- ✅ Support rendering additional views (other than primary view) into a render texture
- ✅ Double buffering support (cameras see the contents of their own render texture from previous frame)
  - 📖 Bevy has view-target ping-pong, prepass double buffers, and enough render-target control to implement explicit camera texture double buffering.
- ✅ Support camera stacking with render order (depth)
	- ✅ Multiple cameras must be able to render into the same texture
	- ✅ Order must be able to be defined (e.g. with depth value)
	- ✅ Must support viewport configuration (rendering to a sub-section of the render texture)
	- ✅ Must support different clear methods (none, depth only, color/skybox)

### Reflection probes
**Required:**
- ✅ Reflection probe that affects specular lighting on materials (notably PBS materials)
- ✅ Varying smoothness support stored in mip maps
- ⚠️ Real time reflection probe rendering
	- 🛑 Support for time slicing when rendering (doesn't need to be baked in as long as we have control where individual phases of render occur)
	  - 📖 Bevy can filter generated environment maps at runtime, but a full built-in scene-capture reflection probe with time slicing was not found.
- ✅ Baked reflection probes
	- ✅ Using cubemap with mipmaps storing different smoothness levels
- ⚠️ Rendering reflection probe to cubemap asset
  - 📖 Bevy has cubemap assets, generated environment maps, and render-to-texture cameras. Rendering all six cubemap faces as a probe capture would need custom camera/render graph code.
- ⚠️ Control over when is reflection probe rendered
  - 📖 Control is possible for custom cubemap capture code, but not exposed as built-in probe update modes.

## Scene model API
Since FrooxEngine has its own scene model, the ideal state for the renderer is to be minimalistic and hold only data absolutely necessary for the rendering itself. If needed, we can have our own code to hold this data on top of the renderer if it doesn’t naturally hold it between frames.

**Required:**
- ✅ Mechanism to filter which entities are rendered for each camera
  - 📖 Bevy has `RenderLayers`, visibility components, camera targets, and per-view visible entity lists.

**Ideal:**
- 🛑 Flat scene description with no transform hierarchy - we submit 4x4 matrices for each entity to be rendered
  - 📖 Bevy is ECS/hierarchy based and retains scene state.
- ⚠️ Submitted poses are in world space and can be shared between multiple cameras/views
  - 📖 Bevy computes/extracts `GlobalTransform` world-space data, but this is not a flat externally-submitted render list API.
- ✅ Renderer performs camera frustum culling & sorting

**Possible alternatives:**
 - ✅ Camera frustum culling & sorting is performed on engine side & fully sorted list of “render commands / draw calls” is submitted to the renderer for each camera / view
   - 📖 Bevy can bulk-extract scene data into its render world and queue per-view render phases; a custom integration could submit higher-level ECS/render data instead of IPC drawcalls.

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
- ⚠️ API to upload arbitrary texture data to the GPU provided in a byte buffer
	- ⚠️ Support for uploading texture sub-regions
	  - 📖 Low-level wgpu `RenderQueue::write_texture` supports this; Bevy's high-level `Image` prepare path updates/reuses whole textures by descriptor.
	- ⚠️ Support for specifying max mip level to currently use
	  - 📖 Texture descriptors and sampler LOD clamps exist. Turnkey progressive high-level mip upload/use would need custom asset or render code.
	- ✅ Support for dynamic updates (textures can be updated anytime - important for procedural textures)
- ✅ Support for common compressed formats (depending on HW)
	- ✅ Uncompressed formats (e.g. RGBA32/ARGB32)
	- ✅ Block compressed formats (BCx, ETCx, ASTC...)

**Ideal:**
- ⚠️ Uploads can be fully done from background threads - engine takes care of any necessary synchronization
  - 📖 Asset loading is async and GPU preparation is synchronized through the render app, but arbitrary renderer-owned background texture uploads would need custom integration.

### Meshes
**Required:**
- Supported vertex attributes
	- ✅ Positions
	- ✅ Normals
	- ✅ Tangents
	- ⚠️ UV’s
		- 🛑 At least 8 channels
		  - 📖 Bevy's built-in mesh/PBR/glTF support exposes `ATTRIBUTE_UV_0` and `ATTRIBUTE_UV_1`; custom vertex attributes are possible but not StandardMaterial/glTF parity.
		- ⚠️ 2-4 dimensions per each channel
		  - 📖 Built-in UVs are `Float32x2`; custom attributes can use other vertex formats with custom pipelines.
	- ✅ Colors
- Supported topologies
	- ✅ Triangles
	- ✅ Points
- ✅ Skinned mesh data
	- ✅ Support binding 1-4 bone transforms to each vertex
	- ⚠️ Support blendshape data in some form
		- ✅ Positions, normals, tangents
		- ⚠️ Support more than one frame for blendshape (progression of blendshape goes through multiple frames)
		  - 📖 Multiple morph targets and animated weights are present; explicit multi-frame blendshape semantics are not.

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
- ✅ Keyboard support (including Unicode)
     - ✅ Individual key press events
     - ✅ Type delta
     - ✅ IME composition
- ✅ Mouse support
     - ✅ At least 5 mouse buttons
     - ✅ Scroll wheel (including horizontal ones)
     - ✅ Absolute position
     - ✅ Relative delta
- 🛑 VR input support
     - 🛑 Access state of all controller buttons/elements
     - 🛑 Haptics support
     - 🛑 Access to full skeleton of the hand (for supported controllers)
- ⚠️ Touch support
     - ✅ Multi-touch
     - ⚠️ Ideally have access to other properties like pressure
       - 📖 Touch force is exposed where the platform provides it, including iOS/Windows pressure-style data and Apple Pencil altitude angle.

**Nice to have:**
- ✅ Gamepad support
- ⚠️ Pen/Stylus support
     - ⚠️ Pressure, tilt etc...
       - 📖 Some stylus-like pressure/altitude data is available through touch force, but no full pen/stylus API with tilt/buttons was found.

## Other "nice to have"
- 🛑 Spout support
- 🛑 Video encoding support (e.g. FFmpeg/NVENC integration and so on)
