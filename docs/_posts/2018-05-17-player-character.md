---
layout: default
title: (Godot) Player Character
---
Continuing with my work on characters in <a href="https://godotengine.org/">Godot</a>, I've spent the last week working on interactions for the player character.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">I added new art assets that are more consistent with the look of the customizable characters. I added some player character actions. This creates a simple and growing village demo in <a href="https://twitter.com/hashtag/godotengine?src=hash&amp;ref_src=twsrc%5Etfw">#godotengine</a>.<a href="https://t.co/Szu73sCnoF">https://t.co/Szu73sCnoF</a> <a href="https://t.co/TjxK4R9W3w">pic.twitter.com/TjxK4R9W3w</a></p>&mdash; Guillaume Laforte (@GuillaumeLafor3) <a href="https://twitter.com/GuillaumeLafor3/status/997175613940293632?ref_src=twsrc%5Etfw">May 17, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<h3>Art Style</h3>
As an aside, I've bought a first pack on the Unity Store from <a href="http://syntystudios.com/">Synty Studios</a>. I think the simple polygonal art-style will work well for my paper-doll characters. Adding the various trees and rocks helps quite a bit to fill the environment. Integrating the static meshes from <a href="http://syntystudios.com/">Synty Studios</a> through Blender and into Godot was straight-forward: no modifications were required in Blender, just import as FBX and re-export into COLLADA. In Godot, due to the restriction of my navigation structure, all the buildings and rocks were scaled to be 4 m^2 in the XZ-plane.

<h3>Player Character Action</h3>
This video shows the player character doing a few actions. The NPC can also be seen to be execute a 'move-to' action and a 'plant' action. I've rebuild the AI to use a new more scalable action system.

<h4>Axe</h4>
In the video, you can see the player character swinging an axe. Attaching a tool to the player character is done dynamically using the paper doll technology. I need to figure out a way to do this without having to change the PaperDoll class everytime I want to introduce a new tool!

<h4>Intersections</h4>
I've been intentionally avoiding using the physics engine to do collision detection and ray-cast intersections. I wrote a simple iterative ray-tracing algorithm with my navigation structure to figure out whether the axe hits something when the player character swings it. Using the forward direction from the player character's transform, I loop 200 times, elongating the forward direction vector by 10 cm every time, for a maximum distance of 21 meters. Starting from the player character's origin, I take these 200 positions and find the list of integer-space cells they correspond to. Then, I check if anything resides in those cells (excluding the player character).

When the player swings its axe, I can detect whether the closest object, in front of it, is close enough for the axe to reach. If there is something within range: I need to devise a good way to handle interactions. This is a problem that I did not anticipate. In this demo, I wrapped the possible interactions using custom scripts attached to the buildings/tree, and the 'meta' system on the Object class. I foresee that the number of interactions pairs could grow quite a bit and a more scalable system could be necessary.

<h4>Inventory and Intelligence</h4>
Following the <a href="https://en.wikipedia.org/wiki/Composition_over_inheritance">Composition over Inheritance</a> principle, I've introduced the character inventories (both the player and the NPCs) and their action queue as new C++ classes inheriting the Node: Inventory and Intelligence. When an Actor-derived object wants to access the character's inventory or its AI, it needs to call 'actor->find_node("Inventory")'.

In the case of the NPC, the Intelligence node is extended with a Godot script that enqueues actions: looping a 'move-to' action and a 'plant' action. This script should expand into a simple worker AI for all NPCs. For the player character, the Intelligence node is also extended with a Godot script. This script is responsible for reading the keyboard and mouse events are translating those into actions. Right now, the 'move-freely' action is the default action: it is fed with the relative movements extracted from the keyboard. When the user presses the 'e' key: the 'attack' action replaces the 'move-freely' action; when the user pressed the 'q' key: the 'plant' action replaces the 'move-freely' action.

<h4>Animations</h4>
Last week, I added a simple <a href="https://en.wikipedia.org/wiki/Finite-state_machine">finite state machine (FSM)</a>, written in Godot Script, to the animation player for the paper doll character system. I want to figure out a decent way to introduce expandable variables and animation nodes to the FSM's node-graph.
