We can now combine all of the structures and objects from the previous chapters
to create the graphics pipeline! Here's the types of objects we have now, as a
quick recap:

* Shader stages: the shader modules that define the functionality of the
programmable stages of the graphics pipeline
* Fixed-function state: all of the structures that define the fixed-function
stages of the pipeline, like input assembly, rasterizer, viewport and color
blending
* Pipeline layout: the uniform and push values referenced by the shader that can
be updated at draw time
* Render pass: the attachments referenced by the pipeline stages and their usage

All of these combined fully define the functionality of the graphics pipeline,
so we can now begin filling in the `VkGraphicsPipelineCreateInfo` structure at
the end of the `createGraphicsPipeline` function.

```c++
VkGraphicsPipelineCreateInfo pipelineInfo = {};
pipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipelineInfo.stageCount = 2;
pipelineInfo.pStages = shaderStages;
```

We start by referencing the array of `VkPipelineShaderStageCreateInfo` structs.

```c++
pipelineInfo.pVertexInputState = &vertexInputInfo;
pipelineInfo.pInputAssemblyState = &inputAssembly;
pipelineInfo.pViewportState = &viewportState;
pipelineInfo.pRasterizationState = &rasterizer;
pipelineInfo.pMultisampleState = &multisampling;
pipelineInfo.pDepthStencilState = nullptr; // Optional
pipelineInfo.pColorBlendState = &colorBlending;
pipelineInfo.pDynamicState = nullptr; // Optional
```

Then we reference all of the structures describing the fixed-function stage.

```c++
pipelineInfo.layout = pipelineLayout;
```

After that comes the pipeline layout, which is a Vulkan handle rather than a
struct pointer.

```c++
pipelineInfo.renderPass = renderPass;
pipelineInfo.subpass = 0;
```

And finally we have the reference to the render pass and the index of the sub
pass where this graphics pipeline will be used.

```c++
pipelineInfo.basePipelineHandle = VK_NULL_HANDLE;
pipelineInfo.basePipelineIndex = -1; // Optional
```

There are actually two more parameters: `basePipelineHandle` and
`basePipelineIndex`. Vulkan allows you to create a new graphics pipeline by
deriving from an existing pipeline. The idea of pipeline derivatives is that it
is less expensive to set up pipelines when they have much functionality in
common with an existing pipeline and switching between pipelines from the same
parent can also be done quicker. Right now there is only a single pipeline, so
we'll simply specify a null handle.

Now prepare for the final step by creating a class member to hold the
`VkPipeline` object:

```c++
VDeleter<VkPipeline> graphicsPipeline{device, vkDestroyPipeline};
```

And finally create the graphics pipeline:

```c++
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &pipelineInfo, nullptr, graphicsPipeline.replace()) != VK_SUCCESS) {
    throw std::runtime_error("failed to create graphics pipeline!");
}
```

The `vkCreateGraphicsPipelines` function actually has more parameters than the
usual object creation functions in Vulkan. It is designed to take multiple
`VkGraphicsPipelineCreateInfo` objects and create multiple `VkPipeline` objects
in a single call.

The second parameter, for which we've passed the `VK_NULL_HANDLE` argument,
references an optional `VkPipelineCache` object. A pipeline cache can be used to
store and reuse data relevant to pipeline creation across multiple calls to
`vkCreateGraphicsPipelines` and even across program executions if the cache is
stored to a file. This makes it possible to significantly speed up pipeline
creation at a later time. We'll get into this in the pipeline cache chapter.

Now run your program to confirm that all this hard work has resulted in a
successful pipeline creation! We are already getting quite close to seeing
something pop up on the screen. In the next couple of chapters we'll set up the
actual framebuffers from the swap chain images and prepare the drawing commands.

[C++ code](/code/graphics_pipeline_complete.cpp) /
[Vertex shader](/code/shader_base.vert) /
[Fragment shader](/code/shader_base.frag)