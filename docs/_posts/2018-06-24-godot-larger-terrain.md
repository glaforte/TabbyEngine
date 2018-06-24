---
layout: default
title: Godot - Terrain Tiles
---
Today is Quebec's national holiday. It's a great day for some fun coding! I've been pondering some exploration gameplay and I decided to work on further expansions of the terrain, this week.

<h3>Larger Terrain</h3>
I've reworked the Battlefield and Terrain classes to support multiple terrain tiles. Here's an example with 5 terrain tiles organized in a cross.

<img width='100%' src="../../../assets/godot-larger-terrain-1.PNG"/>

The terrain tiles shown in this screenshot are (-1,0),(0,-1),(0,0),(0,1),(1,0). The creation of these specific tiles is hard-coded for my testing right now. I need to figure out the way to create on-demand! Also, working on this prototype has shown me that it won't be easy to devise an API where the tiles can be efficiently created one at a time.

Each terrain tile is 128m x 128m right now and with a cell dimension of 2m x 2m, this means that each terrain tile hold 64x64=4096 cells. Previously, the Battlefield class had parameters for the number of cells to generate; those are gone.

<h4>Obstacles</h4>
When checking for obstacles and to add new objects in the scene, the code cannot expect a rectangular area anymore. A rectangular area was significantly simpler to handle, but the terrain tiles idea has one interesting simplification: if there is no terrain tile for a given cell, then there is no world at this cell. I was able to remove all the code that clamped cell indices and the code that checked for boundary conditions outside of the terrain tile.

<h4>Cell Spaces</h4>
This is where the complexity and the bugs come in. I've had to introduce a world-space cell index and a local-space cell index. Hence, each cell has two indices to identify it. A tile is also indexed using two integer values. Previously, I used Godot's Point2i structure for all cell indices and that made the code easy to read. Now, Point2i is used for the world-space cell index, for the local-space cell index and for the terrain tile index, which gets a bit confusing in some algorithms like the terrain smoothing code.

There's also a trick that I had not anticipated when it comes to negative tile indices. Naively, you would use a formula such as: 'local-cell-index = world-cell-index / 64' to convert between spaces, but that breaks down when you hit terrain tiles with negative indices. For example, tile (0,0) contains cells (0,0)-(63,63), but tile (-1,0) contains cells (-64,0)-(-1,63).

<h3>Noise</h3>
I've replaced the double-waveform terrain heights by some fractal-Simplex noise. I integrated into Godot the <a href="https://github.com/Auburns/FastNoise">FastNoise</a> library by Auburns. It was very simple to integrate and its API is simple to use. It is released under the MIT license, so that makes it a great choice! I'll have to do further experiments with the different noise functions and I like that the FastNoise library empowers me to do these experiments.
