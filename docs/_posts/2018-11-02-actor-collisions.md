---
layout: default
title: Actor Collisions
---
This blog post covers the past two weeks. Sadly, I caught a cold about 8 days ago and I've been recovering since last weekend. It saddens me that I will probably miss the last decent days for exercising outside of 2018. In my project, I've been working on the game design document and I've decided to take a tangent from my original project idea. I want to eventually build an history-based creative story, but I'm going to build a smaller project first. More details on this later!

<h3>Collisions</h3>
Continuing with my voxel engine's work on voxel collisions, I've focused last week on the collisions between actors. I had received the feedback that the character movement felt restricted when approaching objects and villagers. At the time, the voxel engine recorded the only and only one actor that was present per voxel.

I researched different solutions and decided to change the actor collisions to use an <a href="https://en.wikipedia.org/wiki/Octree">octree</a> to sort and hold the axis-aligned bounding boxes(<a href="https://en.wikipedia.org/wiki/Minimum_bounding_box#Axis-aligned_minimum_bounding_box">AABB</a>) of the actors. I've extracted all the actor collision code out of the voxel engine and into a new facade. The voxel engine still handles AABB collisions with its static objects, since that's optimal. I think the voxel engine has a nice future for all trees, stones, grasses, the ground and walls. Eventually, I think it'll be awesome for doors, traps: items that are not static, but cannot justify the full cost of an actor. This means that my project will have much fewer actors and they can now have different sizes. Two small actors can now interact without a problem within one voxel, and an actor can stick to another actor regardless of the voxel boundaries. It does makes path-finding more complex. I have not yet wrapped my head around the full impact on path-finding.

I'm using the Godot Engine's C++ octree and AABB classes to detect actor collisions, with awesome results. They are fast and were easy to integrate!

<h3>Wolf</h3>
I've introduced the wolf as a new actor in my project. A first enemy! He gets spawned far from the player, typically outside of the loaded terrain. Hence: I've changed the handling of actors so that they can operate on two levels. The set of terrain tiles that have been created and are drawn near the player is called the physical terrain. It defines where physical interactions are possible because the terrain information is available. The wolf actor now spawns outside of the physical terrain. Outside of the physical terrain, the wolf moves following the generated heightmap of the terrain and other high-level features, a.k.a the logical terrain. The transition of the wolf between the loaded terrain and the logical terrain has an extra step: the engine searches for a valid physical position near the wolf's logical position, and teleports the wolf. When inside the physical terrain, the wolf is now physically simulated and obey the same rules as the player.

<img width="50%" src="../../../assets/wolf-1.PNG"/>

<h4>More Logical and Physical</h4>
Similarly to the distinction with the logical and physical terrains, I've decided to split the actor class. The logical actor object remains in memory permanently and includes the position, the AI, the combat function, hit points and inventory. The physical actor is the one that can be turned off when outside of the physical terrain: it contains the sounds, the animations, the collisions, the rendering.

<h3>Human Model and Animations</h3>
One of the business task that I've advanced quite a bit: to remove the assets that I do not have a flexible-enough redistribution license for. That includes the rabbit model and its animations, as well as the mud hut building, the old trees and brown stones. Also, I was using the Mixamo rig and many of its animations for the human model. Mixamo does not give you redistribution rights when you download it: you're allowed to use the models and animations only as final distribution. I've been dabbling in Blender for some time now and I spent the time to set-up a new armature with fewer bones and hooked my human model on it. I've also spent a bit of time with quadifying the human model so that it looks more consistent with the low-poly look of the rest of the project. Then, I wiped all the Mixamo animations and built my own. That's six new animations that I will need to iterate on: idling, walking, hitting, missing, placing, dying!

<img width="50%" src="../../../assets/human-villager-1.png"/>
