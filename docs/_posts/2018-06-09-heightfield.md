---
layout: default
title: (Godot) Heightfield
---
Another hard week at work. I find it surprising how little the company that I joined, cares about on-boarding new employees. Well, this means I'm even more dedicated to continuing improving my game prototype.

<h3>Terrain Improvements</h3>
I've spent some time, last week, introducing a height-field for the previously-flat terrain in my personal project. I find the look interesting. I had to significantly bias the normals on the quads of the terrain cells to get interesting lighting effect on the terrain. I changed the movement and placement algorithms for all the objects and the characters, so that they automatically set the correct height.

<img width='100%' src="../../../assets/godot-heightfield1.png"/>

<h4>Heightfield</h4>
The heightfield is generated using a couple of waveforms, right now, in the X and Y directions. This is a very simple method to get a first terrain with repeated hills. I've also grown the terrain to 64x64. The advantage of waveforms is that generating new tiles of terrain can be done after the initial phase. This could be an interesting development for another week-end!

<h4>Flat Cells</h4>
When placing a building on the cell of a heightfield, I think it is important to have a flat ground. Games have different approaches on how to handle this problem, but all generally agree that you can't just orient the building with the cell. Some games will force the player to create a foundation piece that provides a flat surface to further build on. The simpler solution that I chose is to modify the height-field when the player creates the foundation for a new building.

<h4>Locked Cells</h4>
Creating a mesh for a heightfield, where objects are placed by the player, becomes problematic when the player attempts to place buildings side-by-side. There are some easy options, like forcing all the buildings to be one cell away, or placing the buildings all at the same heights. I wanted more variety in heights of the buildings and I wanted no artificial restrictions. Rather, I devised a system where cells that have a building on it are flagged. When creating the mesh for the heightfield, if a cell is 'locked', its height wins over the neighbors. This creates a flat ground for the building, since all four vertices will be equal height. When two adjacent cells are locked, but have a different height: the two vertices for the adjacency-edge will have multiple heights. That's when my algorithm creates extra quads/triangles, as in this screenshot:

<img width='100%' src="../../../assets/godot-heightfield2.png"/>

<h4>Height Change Tools</h4>
I added a hot-key on the player to enter a new debugging UI mode, where I can change the height and 'locked-status' of cells. In debugging mode, when the user presses '-', '+' or 'L', it changes the cell that is directly in front of the player character. I found it to be an important tool to debug my algorithm!
