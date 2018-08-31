---
layout: default
title: Godot - Voxel Terrain
---
Starting this week, I was back to working full-time on my personal Godot Engine project. I've made a short plan, looking a few months ahead. Working on this personal project is very rewarding after three months of dreadful software development in the medical device industry. My main goal for a couple of week is to scope a co-operative gameplay module. That's a cool project that I've never done before!

<h3>Creative Module</h3>
Before I can experiment with a co-operative gameplay module, I need the players to have something they can do together. My vision for the creative module is quite large and vague. I know that developing a creative module is core to making a great creative game, but this week, I needed to get it started. Inspired by Minecraft, I thought that allowing players to modify the terrain and create structures using block would be a good start. And it would be an interesting challenge to start the co-operative gameplay module.

<h4>Voxelized Terrain</h4>
I started my journey with a 2D heightfield terrain. I hesitated greatly to replace it with a fully voxelized terrain, because I had invested significant time in it. The alternative to a fully voxelized terrain is to create small voxelized areas on top of the 2D heightfield terrain, when the players start building. It sounded simple to me: marking an area of 4x4 cells in the 2D heightfield terrain as voxelized. Then, adding a actor with a 4x4x4 3D voxel structure to the world. Implementing this idea was my first development task on Monday. I added an actor to my world that was an editable column of cubes. 

<img src="../../../assets/godot-voxels-1.PNG"/>

I have since reverted this prototype and have developed a fully voxelized terrain. The failed prototype taught me quite a bit, though! I gained an understanding of the user interface elements that I would need to build. I realized that voxels should not be bigger than 1m^3 and I will need to handle moving the human player character and AI characters that are taller than 1m. From this point on, I did a lot of notebook writing. I decided to keep the 2D heightfield idea for all the high-level terrain components (HLC). In the high-level world, it is much easier to describe the features instead of rasterizing them in a 3D regular structure. The low-level terrain components (LLC) became two simple 3D vectors. One holds the voxel material id for each cell; the other holds a pointer to the Actor* residing in the cell. Any material id of -1 is considered an empty voxel.

<h4>Player Movement and AI Path-Finding</h4>
Upgrading the player movement and path-finding code was tricky: the AI characters and the rabbits need to be aware of the new 3D stuctures. They need to know that they can move into a cell that is one level below or above their current cell. The A* algorithm handles this beautifully, but I did need to introduce a new function in the terrain that returns whether any given cell is empty but just above a solid block. I also needed to introduce a function that checks whether a cell is free as well as the cell above it. I had expected that terrain building and mesh generation would take the most time, but I was wrong: most of my efforts, this week, was spent improving the movement code to fit in this new voxelized world.

<img src="../../../assets/godot-voxels-3.PNG"/>

On Wednesday, you could climb up to the top of this arch as well as go under the arch!

<h3>Co-operative Gameplay Module</h3>
I started my journey into building the co-operative gameplay module without any experience as a network developer. Watching a couple of videos gave me some courage, but I got a lot of confidence after reading the following <a href="http://docs.godotengine.org/en/3.0/tutorials/networking/high_level_multiplayer.html">Godot official documentation page</a>. Following the page allowed me to build the following simple start menu for the project:

<img src="../../../assets/godot-networking-2.PNG"/>

There seem to be two sections to a standard network module: the "joining" section and the "playing" section. The screenshot above shows how easy the Godot Engine makes the "joining" section. The "playing" section is where I've encountered new complexity. I have a new-found respect for game developers that introduce complex co-operative gameplay features in their games. As I progress in sharing the world from the server to every connected players: I realize that all gameplay elements that I've developed. so far, are affected by co-operative gameplay in some form. And they will all need to be reviewed!

