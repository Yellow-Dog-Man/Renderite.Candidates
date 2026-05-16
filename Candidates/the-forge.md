# The Forge Framework
**Repository:** https://github.com/confettifx/the-forge

- 📖Repo is mostly very technical, more information can be found on their [webpage](https://theforge.dev/products/the-forge/)

**License:** Apache 2.0

## AI Use
How AI is actually used in the project, whether AI contributions are allowed, and whether the renderer is human-designed or vibe-coded without technical oversight.

## Basic Overview
**Programming language(s):**

- ⚠️C/C++

**Shading language(s):** ❓

**Rendering paths:**

- 🗨️Forward++
  - 📖They mention this a lot and it's a bit unclear what it means. It seems to be 3D tiled like clustered forward, but needs more research.

**Graphics API's:**

- ✅Vulkan
- ✅Metal
- ✅DirectX 12

**Supported platforms:**

- ✅Windows 10+
- ✅Linux
  - 📖They target the Steam Deck primarily, but SteamOS is Linux, so Linux
- ✅macOS
- ✅Android
- ✅Quest
  - 📖Explicitly called out in repo, so may have special support.
- ✅iOS
- 📖Also supports PS(4/5), Xbox (One/Series), and Switch, for any potential wild future endeavours on console platforms.

**Oldest supported hardware:** ❓
**VR API's:** 

- ✅OpenXR

**Activity:**
- Rough contributor count: 📖8 main contributors, many small PRs (~150 PRs, unknown how many are duplicate contributors)
- Rough user/community size: 📖VERY large - this is a framework that has been implemented in several AAA titles. However, the Discord is behind an approval gate, and I have never been able to get access, so I don't know if they're just very picky and only let a select amount of developers in.

# Existing usage / projects made with this
The Forge has been implemented in a large amount of AAA releases. These include, but are not limited to:

- The official Quest port of Star Wars: Bounty Hunter
- Call of Duty: Warzone Mobile
- Creation Engine (Starfield, TES6, etc.)
- No Man's Sky
- Forza Motorsport
- Hades

These are all proprietary games that don't really talk about their middleware usage, but The Forge Framework is essentially a "pick what you need" framework for putting together a AAA-quality renderer. 

# General notes
Anything noteworthy that's not related to the any of the requirements directly should be added to this section.

## Positive highlights

This is a massively adopted AAA-grade renderer and could open up a large amount of rendering possibilities for us in the future, especially as this is backed by an entity that works with AAA devs on optimization.

## Potential concerns

There is nearly no public-facing documentation and the official Discord server seems really hard to get into. They have a pre-screening application thing that uses the Discord awaiting approval thing, and I have sat there for over a month before with no response or reachout.

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
- ❓ Allow implementation of dynamic VR/desktop switching

**Ideal:**
- ❓Single-pass stereo rendering
- ❓Canted displays rendering support (e.g. Pimax)
- ❓Foveated rendering support

## Render pipeline
**Required:**
- ❓General performance on par or better with current Unity renderer
- ❓Some level of control over graphics pipeline rendering
- ❓Ability to control the order of rendering and sorting
		- ❓ **Ideal:** Ability to create a “group/batch” of render entities that are rendered at once as an unit and have their own internal sorting order

- ❓ Stencil buffer support
	- Supported operations must match the currently exposed ones through materials
		- ❓8 byte integer (0 to 255)
		- ❓Comparison modes:
			- ❓Disabled
			- ❓Never
			- ❓Less
			- ❓Equal
			- ❓LessOrEqual
			- ❓Greater
			- ❓NotEqual
			- ❓GreaterOrEqual
			- ❓Always
		- ❓Stencil Operations
			- ❓Keep
			- ❓Zero
			- ❓Replace
			- ❓IncrementSaturate
			- ❓DecrementSaturate
			- ❓Invert
			- ❓IncrementWrap
			- ❓DecrementWrap
		- ❓Read & Write masks
- ❓LOD support
	- ❓Per-render switching between render entities depending on relative size on the screen
	- ❓Ideally blending support (e.g. through material/shader properties)
- ❓Some form of Global Illumination (GI) support
- ❓ GPU instancing
	- ❓ Ideally fully automated from the render entities
	- ❓ Efficiently render large number of entities with the same material & mesh
	- ❓ Should have some form of support for varying material properties (e.g. fetching them from a buffer)
- ❓ Mirror/Portal rendering
	- ❓Ideally should be as efficient as possible for VR - use single pass rather than two separate renders
	- ❓More flexible on render method
		- ❓Skewed matrix with render to texture (current method)
		- ❓Scissor/Stencil is a potential alternative method

**Ideal:**
- ❓HDR display output
- ❓Multiple Window support
- ❓Mesh shaders (with meshlets) pipeline
  - ❓Provide fallback for older GPU's
- ❓Better alpha sorting/blending handling

**Nice to have:**
- ❓Forms of static / dynamic occlusion culling
- ❓Ability to re-render the same mesh multiple times with different materials efficiently
        - ❓Perform frustum / occlusion culling just once
        - ❓For skinned meshes - transform vertices just once
- ❓Reversed floating point depth buffer

## Shader pipeline
**Required:**
- ❓Shader pipeline must be isolated enough (or made to be isolated) so it can be invoked from our own code at runtime to dynamically compile shaders
- ❓Compute Shader support
	- ❓Needs to support wave intrinsics

**Ideal:**
- ❓Slang support
- ❓SPIR-V
- ❓Existing PBR/PBS shaders

## Post processing
To match feature parity, the rendering pipeline needs to support the same/similar post processing filters as our current render currently does.

**Required:**
- ❓Bloom
	- ❓Must support working with HDR values (above 1.0)
- ❓Motion Blur
	- ❓Motion vector based
	- ❓Support camera blur
	- ❓Support object blur
	- ❓Support skinned mesh blur
	- ❓Ideally supported both in VR & desktop, but desktop is sufficient
- ❓Ambient Occlusion
- ✅Anti-aliasing
	- ✅MSAA (given by the rendering path)
- ❓ VR & single pass support

**Nice to have:**
- ✅Screen space reflections
- ✅Some form of screen space realtime GI

**Not required:**
	- ❓Anti-aliasing (these are largely needed for deferred rendering)
	- ❓TAA
	- ❓FXAA
	- ❓CTAA
	- ❓SMAA
	- …

## Rendering components / entities
Note that the structure does not need to match the current ones, but the engine needs to have API's and structures so we can replicate functionality of these. It is *not* required that the render specifically has those entities implemented, but rather that it has features that allow them to be implemented.

### Mesh Rendering
**Must have:**
- ❓Triangle topology
- ❓Point topology
- ❓Some form of submesh support
	- ❓Allow specifying materials for each submesh - the whole mesh is rendered & culled as a unit, but each submesh is its own 
- ❓Shadow rendering support (depending on material support)
	- ❓Single sided
	- ❓Dual sided
	- ❓None
	- ❓Shadow only 

### Skinned Mesh Rendering
These requirements are on top of standard mesh rendering.

**Required:**
- ❓1-4 bone support
- ❓Blendshape support
	- ❓Multi-frame support (each blendshape goes through several frames)
	- ❓Positions, Normals & Tangents
- ❓GPU accelerated for good performance

**Nice to have:**
- ❓More than 4 bones
- ❓Blendshape support
	- ❓UV’s, Colors
	- ❓Ability to cache rarely changing blendshapes to avoid recomputations
	- ❓Ability to only compute affected vertices & skip rest 
- ❓Dual Quaternion Skinning #487

### Lights
**Required:**
- Supported types:
  - ❓Point
  - ❓Spot
  - ❓Directional
- Supported features:
	- ❓Hard & Soft realtime shadows for for all types
	- ❓Multiple instances of each type
	- ❓Light cookies for point & spot lights
- ❓Control over lighting falloff

**Nice to have:**
- ❓RGB light cookies
- ❓Baked shadowmaps
- ❓Control over realtime shadowmap rendering
	- ❓Ideally the pipeline could render point/spot light shadowmaps once each frame and share them for each camera/view that’s rendered that frame
- ❓Realtime area/polygonal lights

### Cameras
**Required:**
- ❓Support rendering additional views (other than primary view) into a render texture
- ❓Double buffering support (cameras see the contents of their own render texture from previous frame)
- ❓Support camera stacking with render order (depth)
	- ❓Multiple cameras must be able to render into the same texture
	- ❓Order must be able to be defined (e.g. with depth value)
	- ❓Must support viewport configuration (rendering to a sub-section of the render texture)
	- ❓Must support different clear methods (none, depth only, color/skybox)

### Reflection probes
**Required:**
- ❓Reflection probe that affects specular lighting on materials (notably PBS materials)
- ❓Varying smoothness support stored in mip maps
- ❓Real time reflection probe rendering
	- ❓Support for time slicing when rendering (doesn't need to be baked in as long as we have control where individual phases of render occur)
- ❓Baked reflection probes
	- ❓Using cubemap with mipmaps storing different smoothness levels
- ❓Rendering reflection probe to cubemap asset
- ❓Control over when is reflection probe rendered

## Scene model API
Since FrooxEngine has its own scene model, the ideal state for the renderer is to be minimalistic and hold only data absolutely necessary for the rendering itself. If needed, we can have our own code to hold this data on top of the renderer if it doesn’t naturally hold it between frames.

**Required:**
- ❓Mechanism to filter which entities are rendered for each camera

**Ideal:**
- ❓Flat scene description with no transform hierarchy - we submit 4x4 matrices for each entity to be rendered
- ❓Submitted poses are in world space and can be shared between multiple cameras/views
- ❓Renderer performs camera frustum culling & sorting

**Possible alternatives:**
 - ❓Camera frustum culling & sorting is performed on engine side & fully sorted list of “render commands / draw calls” is submitted to the renderer for each camera / view

**Not viable:**
 - Individual drawcalls and render commands are proxied from the main process
	- This would occur too much overhead for IPC
	- It’s possible that the renderer works this way on its side and we write code to hold the simplified scene state, but the scene state needs to be submitted in bulk

## Asset/resource support

### Textures
**Required:**
- Texture types
	- ❓2D
	- ❓Cubemap
	- ❓3D
- ❓API to upload arbitrary texture data to the GPU provided in a byte buffer
	- ❓Support for uploading texture sub-regions
	- ❓Support for specifying max mip level to currently use
	- ❓Support for dynamic updates (textures can be updated anytime - important for procedural textures)
- ❓Support for common compressed formats (depending on HW)
	- ❓Uncompressed formats (e.g. RGBA32/ARGB32)
	- ❓Block compressed formats (BCx, ETCx, ASTC...)

**Ideal:**
- ❓Uploads can be fully done from background threads - engine takes care of any necessary synchronization

### Meshes
**Required:**
- Supported vertex attributes
	- ❓Positions
	- ❓Normals
	- ❓Tangents
	- ❓UV’s
		- ❓At least 8 channels
		- ❓2-4 dimensions per each channel
	- ❓Colors
- Supported topologies
	- ❓Triangles
	- ❓Points
- ❓Skinned mesh data
	- ❓Support binding 1-4 bone transforms to each vertex
	- ❓Support blendshape data in some form
		- ❓Positions, normals, tangents
		- ❓Support more than one frame for blendshape (progression of blendshape goes through multiple frames)

### Video Textures
Since Resonite supports video playback and this is handled by the renderer (due to GPU texture resources being updated) so this support will need to be handled by the new renderer as well.

This will most likely have to be implemented - e.g. through doing integration with libVLC.

**Required:**
- ❓Video texture support integrated with the rendering pipeline
	- ❓Ability to control the playback (playing, looping, playback position)
	- ❓Ability to select audio track that’s decoded
- ❓Ability to get raw audio data
- ❓Support for both local file playback & streaming
	- ❓Streaming video files from a web endpoint
	- ❓Supporting live video streams (like rtmp)

**Ideal:**
- ❓Ideally libVLC integration for maximum compatibility
	- Can be potentially other playback engines, but the format support needs to be similar
- ❓HW/GPU decoding support
- ❓Specifying separate video & audio stream URL’s

## Input handling
Since the renderer will be providing the user a window and interfacing with some of the input devices (e.g. VR), we need to be able to proxy various inputs.

If these are not supported, they should be trivial to implement in Phase 3.

**Required:**
- ❓Keyboard support (including Unicode)
     - ❓Individual key press events
     - ❓Type delta
     - ❓IME composition
- ❓Mouse support
     - ❓At least 5 mouse buttons
     - ❓Scroll wheel (including horizontal ones)
     - ❓Absolute position
     - ❓Relative delta
- ❓VR input support
     - ❓Access state of all controller buttons/elements
     - ❓Haptics support
     - ❓Access to full skeleton of the hand (for supported controllers)
- ❓Touch support
     - ❓Multi-touch
     - ❓Ideally have access to other properties like pressure

**Nice to have:**
- ❓Gamepad support
- ❓Pen/Stylus support
     - ❓Pressure, tilt etc...

## Other "nice to have"
- ❓Spout support
- ❓Video encoding support (e.g. FFmpeg/NVENC integration and so on)
