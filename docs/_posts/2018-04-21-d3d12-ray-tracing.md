---
layout: default
title: (3D) DxR - D3D12 RayTracing
---

<h3>DxR - First Impressions</h3>
There's a lot of stuff in DxR, lots of steps to take to get your first simple ray-traced triangle. It also feel quite monolithic - having to prepare all the meshes and their shaders at once, then packaging it and requesting the creation of the top/bottom acceleration structure. This feels like it will be very slow to build dynamic scenes as shown at GDC 2018.

<h3>DxR - First Results</h3>
These images are from the Visualizer, taken today. There are just a dozen meshes total in the Art Gallery scene, but I focused on a large vase where the reflections and shadows converge. I think this image shows the potential for the additional visual quality of ray-tracing reflection rays!

<table>
      <tr>
        <td><b>Rasterized</b></td>
        <td><b>Ray-Traced</b> (1 bounce only)</td>
      </tr>
      <tr>
          <td><img width='100%' src="../../../assets/DxR1-Raster.png"/></td>
          <td><img width='100%' src="../../../assets/DxR1-Reflection.png"/></td>
      </tr>
</table>

<h4>DXIL</h4>
In DirectX 11, each compiled shader stayed independent. That was the case, as well, in DirectX 12. DXIL is a new compiled shader technology released in spring 2017 that allows us to build a library of shaders all within one byte-code. In my professional experience, I can count of my fingers the number of times that a pixel shader worked with more than one vertex shader (or vice-versa). So, grouping the shaders makes sense to me. In the case of ray-tracing: one DXIL library is meant to hold the ray-generation shader, the closest-hit shader, the other hit shaders as well as the miss shader. Are there benefits to using a DXIL library with a graphics pipeline state object?

In DirectX 11, I was using the shader reflection code to automatically generate UI to fill the constants of the vertex and pixel shaders. With this new DXIL format, I couldn't find any such shader reflection code. I mourn that feature loss.

<h4>PayLoad</h4>
The payload is the information transfered from the ray-generation shader to the other shaders. If the closest-hit shader wants to trace more rays, it initializes its own payload. Since the payload is in-out, it allows for some bi-directional information transfer. Does the length, in bytes, of the payload significantly affect performance? For simplicity, in my shader library, I stuck with using only a color value with the payload structure. I used the color components as flags for the input into the closest-hit shader. I kept the idea that the output of the closest-hit shader and the miss shader are the final colors, but I used the alpha component to differentiate between a hit (with an alpha of 1) and a miss (with an alpha of 0).

<h4>Acceleration Structure</h4>
My first idea about the bottom-layer of the acceleration structure was to create one bottom-layer object for each mesh. Then, create one top-layer object for each instance of the bottom-layer objects, introducing their world-transform at the time, but that didn't work with the current DxR Fallback library. On this topic, none of Microsoft's ray-tracing samples show multiple bottom-layer objects being instanced and used by one top-layer object. All the Microsoft ray-tracing samples use an identity transform with just one top-layer object. Are there future plans for the DxR Fallback library to support this?

For my prototype, that meant describing the bottom-layer objects in world-space. There are important performance implications for a non-visualization engine, of having all the bottom-layer objects in world-space. You have to duplicate a lot of data even when objects are instanced in the world. If one object is transform-animated: does the renderer have to re-create the acceleration structure from scratch every frame?

<h4>Hit Groups</h4>
What is the link between bottom-layer objects and the hit-groups? When I introduced the support for multiple materials in my prototype: linking the hit-groups with the bottom-layer objects sorta happened magically. The first bottom-layer object was assigned to the first hit-group. The second bottom-layer object was assigned to the second hit-group and so on.. How can a more advanced acceleration structure connect to the hit-groups?

<h3>DxR - Ideas</h3>
The Tabby Engine is missing quite a few rendering components that affect the quality of the ray-traced rendering: the images above are not gamma-corrected, they would greatly benefit from some anti-aliasing, mip-maps support for the textures, proper PBR metalness to determine the reflection coefficients... More directly related to the ray-tracing: my implementation creates the acceleration structure once and does not recreate it when objects move. Since my prototype is a visualization application, it works, but that's very limiting. When I tried it, re-creating the complete acceleration structure every frame proved to be a massive performance cost. Are there cheaper options to support moving objects?
