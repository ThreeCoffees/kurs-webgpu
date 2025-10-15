## What is WebGPU?
**WebGPU** is as described in the specification *"an API that exposes the capabilities of GPU hardware for the Web"*.

This means that WebGPU is **not** a particular library or a set of bindings for a particular language or a specific implementation. It is simply a *"language"* we can use to speak to the GPU.

It is a successor to WebGL, which was based on the OpenGL API, but differs from it greatly. It bares more resemblance to modern graphics apis such as DX12, Metal or Vulkan.

WebGPU is very young and currently widely unsupported. *Chrome* has supported it the longest and is by far the most reliable. *Firefox* has recently (this summer) joined the club with their implementation, that is usable, but lacks some features. Native apps are supported through *"wgpu"* which is the same implementation that *Firefox* uses.
## Basic elements
### Adapter
The adapter is used to access the capabilities of the user's GPU such as optional features or larger limits. It's used to request the device.
### Device
The device is used to do basically everything. It is requested from an adapter and provides the capabilities specified inside of it.
### Pipeline
A pipeline contains information on the vertex and fragment shaders that will get executed alongside resources they will be able to access such as vertex buffers and bind groups. 

This information is only on their layouts, their values will be set later during a render pass.

The creation or a pipeline is an expensive process and will usually be done during the initialization.
### Shader
A shader is code being executed on the GPU. There are many languages such as glsl, hlsl or slang. WebGPU uses wgsl — WebGPU shading language.

WebGPU supports vertex shaders, fragment/pixel shaders and compute shaders. Other types include geometry shaders, tessellation shaders, and mesh shaders.
#### Vertex shader
A vertex shader takes in vertex information (position, color, normals etc.), does calculations on them and every 3 vertices rasterizes the triangle and calls the fragment shader for it.

The built-in values for this shaders are:
* inputs:
  * **vertex_index** *(u32)* — index of the currently processed vertex.
  * **instance_index** *(u32)* — index of the currently processed instance.
* outputs:
  * **position** *(vec4<f32\>)* — clip space position of the current vertex using homogeneous coordinates.
#### Fragment shaders
A fragment shader runs for each potential pixel of the render. This includes pixels that for example get covered up in the later stages or get discarded for other reasons. 

It takes interpolated data from the vertex shader and returns a set of colors and a depth value, that will be written to the provided render pass attachments.

The built-in values are:
* inputs:
  * **position** *(vec4<f32\>)* — position of the fragment.
  * **front_facing** *(bool)* — whether the fragment is on the front face or back face.
* outputs:
  * **frag_depth** *(f32)* — updated depth of the fragment.
### Command Queue
Because the CPU and GPU run in parallel and communication between them takes a long time the commands are stored in a queue and sent all at once. 
### Render Pass
A render pass is a list of commands that set the necessary data and execute the shaders. It also contains data on the attachments and how they're treated.

## Hello world! triangle
### Project setup:
Create a new *vite.js* project by calling `npm create vite@latest`. Remove all unnecessary code.

For Typescript to work correctly:
* In `package.json` add `"@webgpu/types": "^0.1.64"` to `devDependencies`.
* In `tsconfig.json`add `"@webgpu/types"` to `compilerOptions.types[]`.

After running `npm i` auto-completion should now work. 

In order to run the app use `npm run dev`.

### Necessary code:
[Page not found · GitHub · GitHub](https://github.com/ThreeCoffees/webgpu-course/tree/main/01-initialization)
