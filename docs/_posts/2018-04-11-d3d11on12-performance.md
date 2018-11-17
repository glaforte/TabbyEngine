---
layout: default
title: (3D) D3D11on12 - First Impressions
---
Recently, I've been upgrading the Tabby Engine to catch up on the latest technologies. My goal is eventually to complete a rendering pipeline that uses the new DirectX RayTracing technology, recently at GDC 2018 by Microsoft.

I started coding the Tabby Engine in 2012 and at the time, DirectX 11 was the cool new technology out there. In order to focus on getting to DirectX RayTracing faster, I've decided to avoid a long re-write of the Tabby Engine's rasterization engine. Rather, I was happily surprised to find out that Microsoft invested in writing a DirectX 11 wrapper on top of DirectX 12: <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/dn913195(v=vs.85).aspx">D3D11on12</a>. Microsoft wrote only one sample showing off this interesting advancement. And there are very few web-pages about it outside of Microsoft. Microsoft hints that using D3D11on12 is useful to access the Direct2D and DirectWrite APIs - and is otherwise incomplete

That did not stop me and having done my research: I quickly implemented the first step of simply wrapping the whole DirectX11 engine inside a DirectX12 device, with success. The DirectX12 device keeps the responsibility of flipping the swap chain. Being very experienced with Microsoft technologies, the only issue that I encountered was to get the timing of the DirectX11 device Flush correct.

<h3>Quality</h3>

Qualitatively, I get everything in the Visualizer as well as the Tabby Editor to draw perfectly well. The compute shaders are giving great results. The main drawback that I found with D3D11on12 is that the DirectX Debug Layer whines a lot:

<pre><code>D3D12 WARNING: ID3D12CommandList::ClearRenderTargetView: The application did not pass any clear value to resource creation. The clear operation is typically slower as a result; but will still clear to the desired value. [ EXECUTION WARNING #820: CLEARRENDERTARGETVIEW_MISMATCHINGCLEARVALUE]
D3D12 WARNING: ID3D12CommandList::ClearDepthStencilView: The application did not pass any clear value to resource creation. The clear operation is typically slower as a result; but will still clear to the desired value. [ EXECUTION WARNING #821: CLEARDEPTHSTENCILVIEW_MISMATCHINGCLEARVALUE]</code></pre>

Before this change, I would rely on the DirectX Debug Layer to find most of my issues. I fixed the errors as the DirectX Debug Layer found them, keeping my output log clear. I typically kept the DirectX Debug Layer turned on at all times - except when doing profiling. Right now, with the D3D11on12 Wrapper, I find that I cannot fix the issues found by that DirectX Debug Layer and this annoys me: it means that the team responsible for this technology did not go the extra mile. This brings me to discuss to report on the performance of the D3D11on12.

<h3>Performance</h3>

Running on my development laptop, I've done two tests: before and after adding the D3D11on12 Wrapper: on the same rocket scene with multiple lights. From other previous profiling seessions: I know that this scene is CPU bound when the resolution is at 1024x768. Before adding this change, the Visualizer reported an average of 480 FPS over 1000 frames. After changing to a DirectX12 Device and introducing the D3D11on12 Wrapper, the Visualizer reports an average of 170 FPS over 1000 frames.

I've since hooked-on the Visual Studio profiler on the Visualizer with the D3D11on12 Wrapper. It reports that most of the extra time is spent in the D3D11on12 Wrapper's AcquireWrappedResources function call. This is done once a frame before flipping the back-buffers on the swap-chain. Reading on the MSDN: this function call is pretty opaque and seems to be necessary, which leaves me without a way to improve the performance. The MSDN also mentions that the D3D11on12 wrapper's performance is not optimized.
