---
layout: default
title: Copy-on-Write Technique
---
As an interesting challenge to discover more of the advanced C++11 features, I've decided to work on multi-threading the Tabby Engine. The goal would be to have all the rendering elements done through a job-like engine and grealtly improve its CPU usage.

<h3>Dual-Threaded Application Design</h3>

Most of the multi-threaded 3D applications that I have worked with and developed in, have followed a dual-thread system with a Main thread (or UI thread or Gameplay thread) and a Render thread. To me, that seems like a decent stepping stone to developing a more advanced multi-threaded application.

<h4>Main Thread Responsibilities</h4>
* Handle window events, mouse events, keyboard events.
* Update assets.
* Executes the draw-objective assets: filling up a scene draft that contains a raster item queue.

<h4>Render Thread Responsibilities</h4>
* Receive the scene draft.
* Convert the scene draft into d3d12 command lists.
* Execute the command lists.

<h3>Inter-Thread Communication</h3>

I've experienced few commonalities in the communication channel in various applications.

At Eidos, using the Unreal Engine 3, the communication channel was a queue of operations. Both the Main thread and the Render thread would hold entire copies of the world and when the Main thread changed its copy of the world: it would add to the queue, a small structure describing the differences to apply. This felt more like a network protocol with an enumerated type and a set of structures were defined. The Main Thread would fill up the queue and the Render Thread read it as fast as it could. This method requires writing a lot of code and it duplicates a large amount of information between the two threads.

<h4>Copy-on-Write Technique</h4>
At CMLabs, we had success using the <a href='https://en.wikipedia.org/wiki/Copy-on-write'>Copy-on-write technique</a> for sharing structures, but we barely scratched their multi-threading potential. The advantage of this technique is that a static 3D structure can be shared without duplication between the Main thread and the Render Thread. Whenever the Main thread decides to modify a 3D structure, it clones it and can write to it without affecting the copy of the 3D structure currently held by the Render thread. A key needs to be added to all copy-on-write structures so that they can be matched after the cloning. In a future frame, when the Render thread receives the cloned 3D structure: it reads this key and can retrieve its matched DirectX cached data. This avoids re-allocating DirectX buffers. A draw-back of the Copy-on-write technique is that polymorphism is difficult to implement.

<h4>Copy-on-Write Structures</h4>
In the Tabby Engine, I want to keep the assets as Main thread-only structures. They are quite tied with Qt and the Declarative UI elements. They're not great for a communication channel that includes cloning. Instead of holding on to their data as members, I've changed them to contain a copy-on-write reference to an equivalent data structure that belongs to a new, simple layer of data structure. The Hook data structures are meant to be used with the copy-on-write references and exchanged between threads.

<h3>Results</h3>

From my post on DirectX 12, on my development laptop, the Visualizer reported an average of 128 FPS over 1000 frames. This was much below the 480 FPS over 1000 frames that I achieved with the DirectX 11 runtime. Introducing the copy-on-write technique into the Tabby Engine and by adding a Render Thread, I was able to reach 450 FPS over 1000 frames, which is a massive improvement - but still not the DirectX 11 performance.

_This is what multi-threading means to me_: you spend 95% of your time designing the communication channel and coding its data structures. Then, you spend 5% of your time implementing the multi-threaded idea. Writing multi-threaded code should not be rushed!

<h4>Details</h4>
* I introduced the Hook-layer structures for the raster objects: HConstantBuffer, HVertexBuffer, HIndexBuffer, HShader, HTexture, HRenderTarget. I disconnected the Direct3D-specific raster objects so that they are contained in a global D3DCache rather than held as user-handles on the agnostic resources. I expected that this would have slowed-down the system by a small amount. This would have been due to an extra search in a map when trying to find the Direct3D-specific object for any given agnostic resource. I used a hash of the id of the agnostic resource as a key. Unexpectedly, the performance went up to 352 FPS over 1000 frames. My investigation shows that this is due to more sharing of resources.
* I changed all the raster item classes to use Hook-layer structures: HClearItem, HCopyItem, HDrawItem, HRayTraceItem, HSwapItem. I did not expect the performance to change much, since I mostly changed straight pointers into copy-on-write references. The performance went up to 400 FPS over 1000 frames. Again, this was due to more sharing of resources.
* I finally achieved a dual-threaded application. I changed the D3DRasterizer class to take in a list of copy-on-write raster-items. I introduced a circular buffer with three queues of raster items. The main thread create a queue of raster items and fills it as it processes the objectives. Once that's complete, the queue of raster items is added to the circular buffer. The D3D rasterizer starts the Render thread and it retrieves the next queue of raster items, when available, and converts it into DirectX command lists. The DirectX command lists are executed on a swap raster-item. The performance went up to 450 FPS. I am surprised: I expected better gains, so I will have to better analyze my profiling data!
