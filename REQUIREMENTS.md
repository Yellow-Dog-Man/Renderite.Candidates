# New Renderer Requirements

## Programming language
We don't have any strict requirements for which programming language would the renderer be written in. The Renderite IPC logic is specifically made to make the interop relatively simple and language agnostic as long as the language supports working and interpreting with memory buffers and few other features.

In other words as long as we can make IPC mechanism to work, we'll adapt to whatever language the renderer uses - the renderer-side logic is intentionally kept as light as possible for this reason.

**Required language features:**
- Working and interpreting raw memory buffers
- Some form of struct / value types representation
- Library to work with shared memory
-- If the language supports interop with C-style ABI libraries, this effectively solves this

**Preferred languages:**
- Rust
- C#

**Acceptable languages:**
- C/C++
- Go
- Others as long as they fullfill the language features

## License
The goal for the new renderer is to be open source. This ensures that we have full control over it (compared to closed source solutions where we are limited by exposed API's) and allows for community forks and contributions.

**Required:**
- Open source
- Allows commercial use

**Ideal:** MIT license

**Acceptable:**
- BSD
- Apache
- Potentially other permissive licenses

**Not acceptable:**
- GPL type licenses
     - These would not be compatible with the rest of our code and 3rd party dependencies

## AI use
Generally we would not accept vibe coded solutions as replacement due to ethical and code quality concerns, especially for a project of this complexity.

The renderer candidate needs to be designed by human designers.

Small AI contributions won't be automatic disqualification, but they can negatively affect the likelyhood of us picking the candidate.

## Rendering path
The rendering path should support large amounts of dynamic lights for all materials efficiently, while also preserving support for MSAA. One such pipeline is a form of Clustered Forward.

This pipeline first collects all lights currently visible in the view. During rendering, shaders access a list of lights within their area and accumulate the lighting information.

**Ideal:** Clustered Forward
**Acceptable:** Variants of Forward+ (should not be pure 2D binning though), potentially alternative pipelines that fit above criteria (please comment if you’re aware of any)
**Not acceptable:**
- Multi-pass Forward (each light adds additional cost)
- Deferred (many lights supported only for specific opaque materials/shaders and has higher overhead)

## Graphics API
**Ideal:** Vulkan
**Acceptable:** 
- DX12 on Windows (if abstracted well enough)
- Metal on OS X / iOS
**Not acceptable:**
- OpenGL
- DX11 (and lower)

## Supported platforms
**Required:**
- Windows (x64)
- Android / Quest
	- Our goal is to support mobile platforms in the future, so the renderer must have support for these
	- This is not required for initial release, but it *must* be on roadmap and in progress

**Ideal:**
- Linux
	- Native Linux support is ideal, but not required as long as it can run under Proton
	- Ideally support both x64 and ARM - this would allow us to natively run graphical client on Raspberry Pi for example
- Mac OS X
	- Native OS X support or even running under emulation is not required, as there's not a huge interest in this platform
- iOS / Vision Pro
	- This is currently not a large goal, but it ideally should be supported as well
	- We'd consider this almost required, but if there's a really good renderer choice that doesn't support this currently, it won't be a full deal breaker - however we might heavily weigh renderers that do support this

## Supported hardware
**Required:**
- At very least support common GPU's released within last 10 years
  - Roughly DX11 capable hardware
  - Fallbacks for older HW are bonus points - do take note of the oldest hardware that's supported

**Ideal:**
- Modern CPU instruction set optimizations

## VR API
**Ideal:** OpenXR
**Acceptable:** OpenVR, Oculus SDK - only if OpenXR is in development and will be added later

## VR Rendering
**Required:**
- Allow implementation of dynamic VR/desktop switching
	- Ideally through easily controlling which render entities render for the VR view and which for the desktop view
	- This would allow rendering some proxy visuals in VR while in desktop mode

**Ideal:**
- Single-pass stereo rendering
	- Exact form isn’t particularly important as long as it works with the whole graphics pipeline
	- The exact form can also be easily changed in the future, as we don’t expose any implementation specifics of single/multipass, so the VR rendering method should be fully abstracted
- Canted displays rendering support (e.g. Pimax)
	- The renderer should support these efficiently, rather than relying on workarounds like reducing FOV
	- Not a hard requirement for the switch, but the rendering pipeline should be able to accommodate for this, at least in the future
- Foveated rendering support
	- Ideally through variable rate shading

## Render pipeline
**Required:**
- General performance on par or better with current Unity renderer
   - Providing some benchmark information to get us rough idea of the performance helps with the choices
   - This might be harder to verify until next phases, but we should look at any possible indicators of performance
   - Look out for potential concerns (e.g. poorly optimized renderers or scenarios)
- Some level of control over graphics pipeline rendering
	- E.g. so we can implement gaussian splat rendering in the right parts of the pipeline
- Ability to control the order of rendering and sorting
	- Used to replicate Render Queue behavior
		- Essentially allows rendered entities to be put into a “bucket” which is then sorted and rendered
	- Must have enough expressivity so we can also implement sub-ordering within Render Queue
		- This way we can implement “SortingOrder” for rendering
		- This is typically used for UIX canvases to ensure rendering order of many elements
		- **Ideal:** Ability to create a “group/batch” of render entities that are rendered at once as an unit and have their own internal sorting order
			- This would be used to improve UIX rendering, which is typically composed from multiple mesh renderers that need to be rendered in particular order
		- Some form of efficient opaque & transparent pass rendering

- Stencil buffer support
	- This is required as we have shaders that use these
	- UIX also heavily uses stencil buffers for its operation
	- Supported operations must match the currently exposed ones through materials
		- 8 byte integer (0 to 255)
		- Comparison modes:
			- Disabled
			- Never
			- Less
			- Equal
			- LessOrEqual
			- Greater
			- NotEqual
			- GreaterOrEqual
			- Always
		- Stencil Operations
			- Keep
			- Zero
			- Replace
			- IncrementSaturate
			- DecrementSaturate
			- Invert
			- IncrementWrap
			- DecrementWrap
		- Read & Write masks
- LOD support
	- Per-render switching between render entities depending on relative size on the screen
	- Ideally blending support (e.g. through material/shader properties)
- Some form of Global Illumination (GI) support
	- E.g. light probes
	- Must be accessible at runtime to modify the data to allow baking from the runtime for any baked solutions
	- Real-time solutions (e.g. screenspace) are also acceptable
- GPU instancing
	- Ideally fully automated from the render entities
	- Efficiently render large number of entities with the same material & mesh
	- Should have some form of support for varying material properties (e.g. fetching them from a buffer)
- Mirror/Portal rendering
	- Ideally should be as efficient as possible for VR - use single pass rather than two separate renders
	- More flexible on render method
		- Skewed matrix with render to texture (current method)
		- Scissor/Stencil is a potential alternative method
			- However this needs to mesh with the material support for this (e.g. refraction effects)

**Ideal:**
- HDR display output
	- Not strictly required for initial swap, but should be supported at some point
	- Especially once HDR displays become more common in VR headsets
- Multiple Window support
        - We should be able to render multiple views to multiple windows
        - Will allow Resonite to provide better desktop experience with multi-window setup
        - Not strictly required, but should be feasible to implement at some point
- Mesh shaders (with meshlets) pipeline
        - More modern than vertex -> geometry / tessellation -> fragment pipeline
        - Can be faster too
        - Provide fallback for older GPU's
- Better alpha sorting/blending handling
    - Any approaches for rendering alpha blended objects that avoid sorting issues (transparent objects popping in front due to being sorted by their center)

**Nice to have:**
- Forms of static / dynamic occlusion culling
        - Not required for compatibility
        - If the renderer already provides static / dynamic occlusion culling mechanisms, it's bonus points
        - If this functionality is lacking, ideally it should be able to be added at some point in the future
- Ability to re-render the same mesh multiple times with different materials efficiently
        - Perform frustum / occlusion culling just once
        - For skinned meshes - transform vertices just once
        - This will allow us to implement material stacking efficiently
        - If not supported, we can emulate behavior by submitting the same mesh multiple times with the same pose, but this will be less efficient
- Reversed floating point depth buffer
        - Nice to have for better precision, but not required
        - Bonus points if supported

## Shader pipeline
**Required:**
- Shader pipeline must be isolated enough (or made to be isolated) so it can be invoked from our own code at runtime to dynamically compile shaders
- Compute Shader support
	- Needs to support wave intrinsics
	- This will be used to re-implement (port) a number of features
		- SH2 calculation
		- Gaussian splat rendering

**Ideal:**
- Slang support
- SPIR-V
- Existing PBR/PBS shaders
	- This will reduce a lot of workload porting PBS shaders by basing them on the existing implementation

**Acceptable:**
Other common shading languages are acceptable too as long as they fit “must-have’s” - we will be abstracting the pipeline away and not exposing the language directly at the moment

## Post processing
To match feature parity, the rendering pipeline needs to support the same/similar post processing filters as our current render currently does.

**Required:**
- Bloom
	- Must support working with HDR values (above 1.0)
- Motion Blur
	- Motion vector based
	- Support camera blur
	- Support object blur
	- Support skinned mesh blur
	- Ideally supported both in VR & desktop, but desktop is sufficient
- Ambient Occlusion
	- Specific type isn’t strictly required
- Anti-aliasing
	- MSAA (given by the rendering path)
- VR & single pass support
-- Unless specified otherwise, the post processing effects must support VR & single pass rendering properly

**Nice to have:**
- Screen space reflections
	- Those we can potentially drop as they’re mostly experimental
- Some form of screen space realtime GI

**Not required:**
	- Anti-aliasing (these are largely needed for deferred rendering)
	- TAA
	- FXAA
	- CTAA
	- SMAA
	- …

## Rendering components / entities
Note that the structure does not need to match the current ones, but the engine needs to have API's and structures so we can replicate functionality of these. It is *not* required that the render specifically has those entities implemented, but rather that it has features that allow them to be implemented.

### Mesh Rendering
**Must have:**
- Triangle topology
- Point topology
- Some form of submesh support
	- Allow specifying materials for each submesh - the whole mesh is rendered & culled as a unit, but each submesh is its own 
	- Doesn't need to be supported "natively", as long as the render pipeline has enough expressiveness to replicate this behavior
- Shadow rendering support (depending on material support)
	- Single sided
	- Dual sided
	- None
	- Shadow only 

### Skinned Mesh Rendering
These requirements are on top of standard mesh rendering.

**Required:**
- 1-4 bone support
- Blendshape support
	- Multi-frame support (each blendshape goes through several frames)
	- Positions, Normals & Tangents
- GPU accelerated for good performance

**Nice to have:**
- More than 4 bones
- Blendshape support
	- UV’s, Colors
	- Ability to cache rarely changing blendshapes to avoid recomputations
	- Ability to only compute affected vertices & skip rest 
- Dual Quaternion Skinning #487
        - Not a requirement, but bonus points

### Lights
**Required:**
- Supported types: Point, Spot, Directional
- Supported features:
	- Hard & Soft realtime shadows for for all types
	- Multiple instances of each type
	- Light cookies for point & spot lights
- Control over lighting falloff
        - Enough control over lighting model so we can implement custom falloff if needed
        - If the falloff curve differs from the current Unity renderer, we must be able to at very least provide "legacy" option
        - Ideally the falloff would be user configurable for creative control

**Nice to have:**
- RGB light cookies
- Baked shadowmaps
- Control over realtime shadowmap rendering
	- Ideally the pipeline could render point/spot light shadowmaps once each frame and share them for each camera/view that’s rendered that frame
- Realtime area/polygonal lights

### Cameras
**Required:**
- Support rendering additional views (other than primary view) into a render texture
- Double buffering support (cameras see the contents of their own render texture from previous frame)
	- Doesn’t need to be baked in, but must be possible to implement on our end
- Support camera stacking with render order (depth)
	- Multiple cameras must be able to render into the same texture
	- Order must be able to be defined (e.g. with depth value)
	- Must support viewport configuration (rendering to a sub-section of the render texture)
	- Must support different clear methods (none, depth only, color/skybox)

### Reflection probes
**Required:**
- Reflection probe that affects specular lighting on materials (notably PBS materials)
- Varying smoothness support stored in mip maps
- Real time reflection probe rendering
	- Support for time slicing when rendering (doesn't need to be baked in as long as we have control where individual phases of render occur)
- Baked reflection probes
	- Using cubemap with mipmaps storing different smoothness levels
- Rendering reflection probe to cubemap asset
	- Doesn’t need to be handled by the probe entity itself, but there needs to be pathway to do this render and obtain the cubemap texture data
- Control over when is reflection probe rendered
        - We need to be able to implement "OnChanges", "Every frame" and "Time sliced" modes
        - If the renderer has control over when the render is triggered, it'll be sufficient - we can control this from FrooxEngine side

## Scene model API
Since FrooxEngine has its own scene model, the ideal state for the renderer is to be minimalistic and hold only data absolutely necessary for the rendering itself. If needed, we can have our own code to hold this data on top of the renderer if it doesn’t naturally hold it between frames.

**Required:**
- Mechanism to filter which entities are rendered for each camera
	- This will be used to implement various mechanisms like render transform override

**Ideal:**
- Flat scene description with no transform hierarchy - we submit 4x4 matrices for each entity to be rendered
- Submitted poses are in world space and can be shared between multiple cameras/views
	- This also potentially allows sharing of other elements - e.g. shadowmaps for lights, rather than rendering them for each camera/view
- Renderer performs camera frustum culling & sorting

**Possible alternatives:**
 - Camera frustum culling & sorting is performed on engine side & fully sorted list of “render commands / draw calls” is submitted to the renderer for each camera / view

**Not viable:**
 - Individual drawcalls and render commands are proxied from the main process
	- This would occur too much overhead for IPC
	- It’s possible that the renderer works this way on its side and we write code to hold the simplified scene state, but the scene state needs to be submitted in bulk

## Asset/resource support

### Textures
**Required:**
- Texture types
	- 2D
	- Cubemap
	- 3D
- API to upload arbitrary texture data to the GPU provided in a byte buffer
	- Support for uploading texture sub-regions
	- Support for specifying max mip level to currently use
		- This allows progressive loading - texture can be used for rendering as soon as the first mip levels have been uploaded
	- Support for dynamic updates (textures can be updated anytime - important for procedural textures)
- Support for common compressed formats (depending on HW)
	- Uncompressed formats (e.g. RGBA32/ARGB32)
	- Block compressed formats (BCx, ETCx, ASTC...)

**Ideal:**
- Uploads can be fully done from background threads - engine takes care of any necessary synchronization

### Meshes
**Required:**
- Supported vertex attributes
	- Positions
	- Normals
	- Tangents
	- UV’s
		- At least 8 channels
		- 2-4 dimensions per each channel
	- Colors
		- Specifying color profile for mesh is not necessary, this can be converted from main process to the right color space
- Supported topologies
	- Triangles
	- Points
- Skinned mesh data
	- Support binding 1-4 bone transforms to each vertex
	- Support blendshape data in some form
		- Positions, normals, tangents
		- Support more than one frame for blendshape (progression of blendshape goes through multiple frames)

### Video Textures
Since Resonite supports video playback and this is handled by the renderer (due to GPU texture resources being updated) so this support will need to be handled by the new renderer as well.

This will most likely have to be implemented - e.g. through doing integration with libVLC.

**Required:**
- Video texture support integrated with the rendering pipeline
	- Ability to control the playback (playing, looping, playback position)
	- Ability to select audio track that’s decoded
- Ability to get raw audio data
	- Meaning we handle the playback through our audio system, rather than the playback engine outputting it directly to audio device
- Support for both local file playback & streaming
	- Streaming video files from a web endpoint
	- Supporting live video streams (like rtmp)

**Ideal:**
- Ideally libVLC integration for maximum compatibility
	- Can be potentially other playback engines, but the format support needs to be similar
- HW/GPU decoding support
	- Allows large (4K) videos
- Specifying separate video & audio stream URL’s

## Input handling
Since the renderer will be providing the user a window and interfacing with some of the input devices (e.g. VR), we need to be able to proxy various inputs.

If these are not supported, they should be trivial to implement in Phase 3.

**Required:**
- Keyboard support (including Unicode)
     - Individual key press events
     - Type delta
     - IME composition
- Mouse support
     - At least 5 mouse buttons
     - Scroll wheel (including horizontal ones)
     - Absolute position
     - Relative delta
- VR input support
     - Access state of all controller buttons/elements
     - Haptics support
     - Access to full skeleton of the hand (for supported controllers)
- Touch support
     - Multi-touch
     - Ideally have access to other properties like pressure

**Nice to have:**
- Gamepad support
     - We could move to FrooxEngine side potentially if it doesn't support it
- Pen/Stylus support
     - Pressure, tilt etc...

## Other "nice to have"
There's a few features that we'd consider "nice to have" in the renderer as a bonus points - however they will only mainly matter if the renderer is otherwise equal (or better) than other candidates.

If the renderer has some fundamental problems, having the "nice to have" features will not make us pick it over others.

Most of these we could implement into the renderer at later point when we introduce those features.

- Spout support
- Video encoding support (e.g. FFmpeg/NVENC integration and so on)

## What will be re-implemented/ported
There’s a number of features that we don’t need the renderer to support directly, because they’ll be re-implemented & ported as part of our integration.

- SH2 calculation from cubemap
	- This is done by a compute shader
- Gaussian Splat rendering support
	- This is handled mostly through compute shaders which can be ported
	- We’ll need the render pipeline to be flexible enough to implement this, but we don’t expect this to be implemented
