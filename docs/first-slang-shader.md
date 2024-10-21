# Your first Slang shader

In this tutorial we demonstrate how to write a simple compute shader in Slang that adds numbers from two buffers and writes the results into a third buffer. To start, create a text file named `hello-world.slang` in any directory, and paste the following content in the newly created file:

```hlsl
// hello-world.slang
StructuredBuffer<float> buffer0;
StructuredBuffer<float> buffer1;
RWStructuredBuffer<float> result;

[shader("compute")]
[numthreads(1,1,1)]
void computeMain(uint3 threadId : SV_DispatchThreadID)
{
    uint index = threadId.x;
    result[index] = buffer0[index] + buffer1[index];
}
```

> #### Note ####
> Slang has official language extension support for both [Visual Studio](https://marketplace.visualstudio.com/items?itemName=shader-slang.slang-vs-extension) and [Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=shader-slang.slang-language-extension). The extensions are powered by the Slang compiler to support a wide range of
> assisting features including auto-completion, function signature hinting, semantic highlighting and more.

As you can see, `hello-world.slang` is no different from a normal HLSL shader file. In fact, Slang is compatible with most HLSL code you would write. On top of HLSL, Slang has added many new language and compiler features that simplifies various tasks with shader code, which we will cover in future chapters. For now we will demonstrate one key feature of Slang: cross-compiling to different platforms.

Slang supports compiling shaders into many different targets including Direct3D 11, Direct3D 12, Vulkan, CUDA and C++ (for execution on CPU). You can run `slangc` with the following command line to compile `hello-world.slang` into Vulkan SPIRV:

```bat
.\slangc.exe hello-world.slang -profile glsl_450 -target spirv -o hello-world.spv -entry computeMain
```

If you would like to see the equivalent GLSL of the generated SPIRV code, simply change the `-target` argument to `glsl`:
```bat
.\slangc.exe hello-world.slang -profile glsl_450 -target glsl -o hello-world.glsl -entry computeMain
```

The resulting `hello-world.glsl` generated by `slangc` is shown below:
```glsl
// hello-world.glsl (generated by slangc)
#version 450
layout(row_major) uniform;
layout(row_major) buffer;

#line 2 0
layout(std430, binding = 0) readonly buffer _S1 {
    float _data[];
} buffer0_0;

#line 3
layout(std430, binding = 1) readonly buffer _S2 {
    float _data[];
} buffer1_0;

#line 4
layout(std430, binding = 2) buffer _S3 {
    float _data[];
} result_0;

layout(local_size_x = 1, local_size_y = 1, local_size_z = 1) in;
void main()
{

#line 10
    uint index_0 = gl_GlobalInvocationID.x;
    float _S4 = ((buffer0_0)._data[(index_0)]);

#line 11
    float _S5 = ((buffer1_0)._data[(index_0)]);

#line 11
    float _S6 = _S4 + _S5;

#line 11
    ((result_0)._data[(index_0)]) = _S6;

#line 8
    return;
}
```

As you can see, things are being translated just as expected to GLSL: the HLSL `StructuredBuffer` and `RWStructuredBuffer` types are mapped to shader storage objects and the `[numthreads]` attribute are translated into proper `layout(...) in` qualifier on the `main` entry-point.

Note that in the generated GLSL code, all shader parameters are qualified with explicit binding layouts. This is because Slang provides a guarantee that all parameters will have fixed bindings regardless of shader optimization. Without generating explicit binding layout qualifiers, the downstream compiler in the driver may change the binding of a parameter depending on whether any preceding parameters are eliminated during optimization passes. In practice this causes a pain in application code, where developers will need to rely on run-time reflection to determine the binding location of a compiled shader kernel. The issue gets harder to manage when the application also needs to deal with shader specializations. Since Slang will always generate explicit binding locations in its output on all targets as if no parameters are eliminated, the user is assured that parameters always gets a deterministic binding location without having to write any manual binding qualifiers in the Slang code themselves. In fact, we strongly encourage users not to qualify their Slang code with explicit binding qualifiers and let the Slang compiler do its work to properly lay out parameters. This is best practice to maintain code modularity and avoid potential binding location conflicts between different shader modules.

## The full example

The full Vulkan example that sets up and runs the `hello-world.slang` shader in located in the [/examples/hello-world](https://github.com/shader-slang/slang/tree/master/examples/hello-world) directory of the Slang repository. The example code initializes a Vulkan context and runs the compiled SPIRV code. The example code demonstrates how to use the Slang API to load and compile shaders.