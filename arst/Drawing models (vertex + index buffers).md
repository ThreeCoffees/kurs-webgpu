## Using vertex buffers
In order to draw something more ambitious than a triangle and also reuse the shader for other shapes we need to find a way to send the vertex data to the GPU.

One way of doing that is through the use of vertex buffers. 

```ad-tip

Vertex buffers are not beholden to the same alignment restrictions as other buffers.
```
### Passing vertex information
#### Position
The first attribute we'll be sending will be the position. First, we have to define the input. 
```wgsl
struct VInput {
	@location(0) position: vec2f,
}

@vertex
fn vs(
	v_input: VInput,
) -> @builtin(position) vec4f {
	return vec4f(v_input.position, 0, 1);
}
```

Next we need to create the vertex buffer layout. This will tell the created pipeline how to read the data provided inside the buffer.
```ts
vertex_buffer_layout = {
	arrayStride: 2 * 4, // 2 x float32
	attributes: [
		{
			offset: 0,
			shaderLocation: 0, // @location(number)
			format: "float32x2", // tells the GPU how to interpret
		},
	],
};
```

This layout needs to be passed to the pipeline during creation.
```ts
vertex: {
	module: shader_module,
	buffers: [
		vertex_buffer_layout
	]
},
```

Separately, we need to create the actual buffer that will be set during a render pass. We can also write the necessary data to it.
```ts
vertex_buffer = device.createBuffer({
	size: data.byteLength,
	usage: GPUBufferUsage.VERTEX | GPUBufferUsage.COPY_DST,
});
device.queue.writeBuffer(vertex_buffer, 0, data);
```

The vertex buffer can be set during a render pass.
```ts
pass.setVertexBuffer(0, vertex_buffer);
```
#### Color
in order to pass the color data we need to extend the input struct and add the output to pass the color to the fragment shader.
```wgsl
struct VInput {
	@location(0) position: vec2f,
	@location(1) color: vec4f,
}

struct VOutput {
	@builtin(position) position: vec4f,
	@location(0) color: vec4f,
}

@vertex
fn vs(
	v_input: VInput,
) -> VOutput {
	out: VOutput;
	out.position = vec4f(v_input.position, 0, 1);
	out.color = v_input.color;
	return out;
}
```

The layout will also need to be modified. There are two ways we can do that: as a `float32x4` or a `unorm8x4`.
##### `float32x4`
Directly analogous to the wgsl type. Uses up more space.
```ts
vertex_buffer_layout = {
	arrayStride: 2*4 + 4*4, // 2 x float32 + 4 x float32
	attributes: [
		{
			offset: 0,
			shaderLocation: 0, 
			format: "float32x2",
		},
		{
			offset: 8,
			shaderLocation: 1,
			format: "float32x4",
		},
	],
};
```
##### `unorm8x4`
Uses less space. WebGPU converts to the wgsl type automatically.
```ts
vertex_buffer_layout = {
	arrayStride: 2*4 + 4*1 + 4, // 2 x float32 + 4 x unorm8
	attributes: [
		{
			offset: 0,
			shaderLocation: 0, 
			format: "float32x2", 
		},
		{
			offset: 8,
			shaderLocation: 1, 
			format: "unorm8x4", 
		},
	],
};
```

### Passing instance information
Besides passing the data for individual vertices vertex buffers can also be used to pass data for instances. In order for the stride to advance per instance we need to set the step mode.

This example will use a vec2f position offset.
```ts
instance_buffer_layout = {
	arrayStride: 2*4
	stepMode: "instance" // <- setting the step mode here
	attributes: [
		{
			offset: 0,
			shaderLocation: 2, 
			format: "float32x2", 
		},
	],
};
```

The buffer needs to be created and set analogously to the previous examples. The number of instances to be drawn is set in the draw command.
```ts
pass.draw(vertex_count, instance_count);
```
## Indexed drawing
When rendering larger objects most of the time vertices can be reused. For example using only 8 for the cube instead of the 36 required without reuse.

This is done through **index buffers**.

An index buffer is an array of indexes organizing the vertices into triangles. The order of vertices matters when determining the front/back of the triangle. By default the front face is **counterclockwise**.

```ts
index_buffer = device.createBuffer({
	size: data.byteLength,
	usage: GPUBufferUsage.INDEX | GPUBufferUsage.COPY_DST,
});
device.queue.writeBuffer(index_buffer, 0, data);
```
 
```ts
pass.setIndexBuffer(index_buffer, "uint32");

pass.drawIndexed(index_count, instance_count);
```

```ad-tip
Length of the index buffer must be aligned to 4 bytes. This can cause problems when using *uint16* if the index count is odd.

```
