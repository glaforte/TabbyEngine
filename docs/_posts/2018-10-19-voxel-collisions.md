---
layout: default
title: Voxel Collisions
---
This week was a mixed week. I went to see a play at the <a href="https://centaurtheatre.com/">Montreal Centaur Theatre</a>: <a href="https://centaurtheatre.com/choir-boy.html">Choir Boy</a>. It was an entertaining, if confusing play! That's also how my week felt. I was productive, but I questioned the targets that I wanted to achieve. Rather than advance further on adding gameplay elements, I reviewed the feedback that I received from my last test session and focused on it. I've been told to focus on fewer elements, but to make sure that they are well-done, so I'm taking this advice :)

<h3>Collisions</h3>
For the trees and stones, I'm introducing a new category of objects. They are more similar to the grasses introduced in the past weeks, but they do have collisions that fit within an expanded definition of a voxel. Each voxel can now hold separate collision and visual information. I've replaced the commercial trees and rocks that I had bought from the Unity store, and built some new ones in Blender that would fit the collision shapes that I added to the voxel definition.

<img width="50%" src="../../../assets/new_tree_and_stone.PNG"/>

<h4>Rotations</h4>
The collision space of a block is currently defined by its shape and its rotation. This allows me to have pyramids and slanted blocks that you can walk on, with various orientations. You cannot see the collisions in the project, only feel them by walking around. Authoring the collision shapes and their rotations remained a challenge until I developped a new node for the Godot Engine. The material collision tester tool outputs two JSON strings that I can copy-paste into the material database, you can see them in the following screenshot, in the Godot Engine Inspector.

<img width="100%" src="../../../assets/material-collision-tool.PNG"/>

<h4>Voxel Definition</h4>
This is the current definition of a voxel in my project. 
<pre>union {
    unsigned int full;
    struct {
        unsigned int background : 3;
        unsigned int foreground : 3;
        unsigned int shape : 5;
        unsigned int collision_rotation : 5;
        unsigned int visual_rotation : 5;
        unsigned int material : 11;
    } bits;
} value;
unsigned int data;</pre>

It's starting to be crammed with a lot of details!
* The background enum defines what's outside of the block's shape: is it solid, liquid or air?
* The foreground enum is a bit-flag that defines if the shape at this voxel is used for collisions, visuals or if it is unused.
* Six shapes are currently supported: the inner/outer pyramids, the slanted block, the quarter/half/full cube.
* Twelve rotations are currently supported: various combinations of 90 degree rotations.
* The material IDs are defined in a JSON database. Within the JSON database, the material defines whether a visual mesh should replace rendering the voxel shapes.
* There is one integer per voxel for custom data.

<h3>Hydrology</h3>
This week, I also progressed on the terrain generation algorithm: to make the rivers, lakes and hills more interesting. The rivers are now larger, the lakes are larger, deeper and more broken-looking. The higher cells are also more broken, producing a few cliffs. This gives the terrain a more interesting look, even if somewhat unrealistic!

<img width="100%" src="../../../assets/overlook-no-trees-nor-stone.PNG"/>
