---
layout: default
title: (3D) - OpenSceneGraph First Impressions
---
<img src='../../../assets/osg64.png'><br>
After a short stay at Eidos Montreal, I took some time off and worked on personal projects and business development. In October 2012, I joined <a href='https://www.cm-labs.com/'>CMLabs</a>, a small simulation company based in the Old Port of Montreal. CMLabs' main business are port, offshore simulators and reliable dynamics. Graphics are not CMLabs' money maker, so their investment in their graphics engine has been minimal, so far. Many years ago, they decided to use the <a href='http://www.openscenegraph.org'>OpenSceneGraph</a> library to get a step up. After a couple of months of using this rendering engine, here are my thoughts...

<h3>Technological Level</h3>
The OpenSceneGraph project has a large amount of features. Robert Osfield and his team have done a good job of keeping this project up-to-date with the more flashy advancements in rendering technologies. I was happily surprised to see support for occlusion queries, geometry shaders and rendering on a separate thread. Overall, the technological advancement level is good and the community has been patching in features such as tessellation shaders. In terms of pure technological advancement, I believe the OpenSceneGraph project is on-par with the modern rendering engines that I have used in the video games industry.

<h3>Materials</h3>
Flashy features, though, are not the bread-and-butter of a rendering engine. I find that the material system in OpenSceneGraph is very basic. Some important features that I took for granted at Eidos and Feeling Software are missing, such as normal maps, multiple light's shadows, shader caching, generating shaders depending on the material demanded by the mesh. Instead, I get the feeling that the fixed-function pipeline is meant to be used for all materials.

<h3>Modern OpenGL</h3>
The use of glBegin and glEnd in some of the code is quite perplexing. Vertex buffer objects have been the norm in DirectX for many years and the OpenGL rendering engine that I wrote for Feeling Software only supported vertex buffer objects. A naive look at the osgParticles module shows a well-designed, particle system module. The implementation of the osgParticle module, though, badly needs a refresh to modern techniques. The implementation of the osg::Geometry class contains a "slow path". Once you realize that there exists a slow and a fast path: should you not write a processing/command-line tool that ensures everyone is using the fast path?

<h3>Threaded Rendering</h3>
The threading method used in OpenSceneGraph is simple and effective for mostly static scenes. The rendering thread works in parallel with the system thread. In real-time, the system thread can modify the values of transforms, colors and such. In order to add or remove nodes from the OpenSceneGraph hierarchy: the system thread does need to stop the rendering thread. Since CMLabs believes in keeping 60 fps at all times: that causes a very important issue for somewhat dynamic scenes. I would not trust OpenSceneGraph to deal with streaming terrain tiles, like we did on Farcry: Instinct, eight years ago.
<p>I have also discussed multi-threaded rendering before, on this blog. I believe that multi-threaded rendering will be key to having great graphics on moderns PCs and future consoles. Attempting to multi-thread the rendering in OpenSceneGraph would be a colossal challenge, since all the data is contained in one hierarchy. From my work at Eidos Montreal: multi-threaded rendering works well pretty much only on flat lists of independent data to process.</p>

<h3>Conclusion</h3>
For an academic group or for a small company whose main focus isn't graphics: OpenSceneGraph is a great choice. It provides decent graphics, export plug-ins and has a simple-enough API. For a company with growing graphical demands: in terms of performance and material quality: there are better options. The challenge of building a top-notch graphical engine on top of OpenSceneGraph will be a very interesting one.