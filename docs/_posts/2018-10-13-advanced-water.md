---
layout: default
title: Advanced Water
---
This was a short week of work for me. The vacation of the past couple of weeks ended with a long week-end. I did some coding and some 3d modeling during the vacation, but mostly it was spending time with my family!

<h3>Hydrology</h3>
Rivers were very important for the early tribes of humans. Today, they matter for trade and logistics, even if their importance on our food production has diminished. I've been researching different ways to produce intelligent rivers since the beginning of the project. During my vacation, I rediscovered <a href="https://azgaar.github.io/Fantasy-Map-Generator/">Azgaar's Fantasy Map Generator</a> and it gave me the inspiration that I was missing!

My improved map generator creates a Voronoi graph from random positions in the world. The random positions are generated using cellular noise. The Voronoi graph generates terrain cells. It produces one height for each cell using two different frequencies of cellular noise. A low frequency is used to generate an island feel. A higher frequency is used to break any visible patterns from the low frequency. The Voronoi graph means that each cell knows which other cells are its neighbors: this is an information that my previous map generation did not have. The neighbors are used to compute the hydrology: each cell drains its rainfall unto the lowest of its neighbors. Also, if a cell's height is lower than any of its neighbors, it will accumulate water until it can drain unto a neighbor: this is essentially a lake.

There's a lot more work that I want to do to improve the terrain generation, including curving and enlarging the rivers, but I was happy to have something to start implementing gameplay features with.

<img src="../../../assets/godot-map-5.PNG"/>

<h3>Underwater</h3>
The addition of rivers was important to me, but I underestimated the impact it would have on my project. It affects most elements of the gameplay. What happens when the user wants to put a block of wood underwater? How can the player character jump over the river? What happens when a rabbit dives in the ocean? Where can I spawn the player's home with a guarantee to be above water? Turns out it is quite a bit of work to adapt everything into a decent package!

<h4>Underwater Camera</h4>
I added code to detect when the player's camera went underwater. When that happens, I duplicate the World Environment and set it to the camera with some tweaks: I add fog and reduce the view distance considerably. This gives me a decent effect that tells the user quickly that he is underwater.

<h4>Underwater Biome</h4>
I made a plan on Wednesday evening to quickly implement some ideas related to fishing. Honestly, I stumbled and I was able to accomplish only part of what I wanted. I had to improve my voxel engine to allow for water and a plant in the same voxel. This means that the engine now supports slanted blocks underwater. I created a first fish model and give it the same behaviour as the rabbit, for now.

<img width="49%" src="../../../assets/underwater-2.PNG"/><img width="49%" src="../../../assets/fish-1.jpg"/>

