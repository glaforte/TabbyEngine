---
layout: default
title: (Godot) Character Variety
---
Following <a href='http://www.tabbycoder.com/2018/05/08/godot-paper-doll.html'>my previous post</a> on 3d paper dolls in <a href="https://godotengine.org/">Godot</a>, I've expanded my prototype with some interesting results, in terms of simple character variety and integrating more <a href="https://www.mixamo.com">Mixamo</a> animations.

<h3>Character Variety</h3>
The idea behind this second technology demo is to use the paper-doll in a village. In this image, you can see the player's character in the bottom-center. The village is represented by a central building. There are 10 other non-player characters (NPCs) on a small area (16m x 12m).

<img width='100%' src="../../../assets/godot-paperdoll-2.PNG"/>

<h4>Paper Doll</h4>
It was pleasant, in the Godot Editor, to build three different options for skin shades and four different options for clothing color. In the C++ code, I've added a Paperdoll class, based on top of my Actor class. The Actor class is the one that handles the registration of the Actor's position with the Navigation structures. I've designed the Paperdoll class to have a simple API for selecting options for clothing style, clothing color and skin shade. Since the three categories can be mixed-and-matched, this yields 36 possibilities for the look for the characters.

<h4>Character Behavior</h4>
The NPCs run a simple AI routine: they move towards a random cell on the field. They avoid collisions with each other and other obstacles. When they move close enough to their goal, they kneel, plant a seed, they stand up and select a new random cell.

Using Godot Script to build a simple animation graph challenged my scripting ability. I discovered that you can easily have Singletons in Godot Script, but they need to extend the Node class and also to be added to the AutoLoad list in the Project Settings.

<h4>Navigation</h4>
I'm using the terrain and navigation technology that I wrote for a 2D puzzle game. I've changed the navigation technology where actors moved in integer-space. In this technology demo, they move in real-time and in floating-space. When moving on a 3D board, in integer-space: the Actor objects would simply teleport from the middle of one cell to the next. In this technology demo, the Actor objects move in real-time, at the <a href='https://en.wikipedia.org/wiki/Walking'>average human walking speed of 1.4 m/s</a>. In order to provide a better-looking animation: I change the direction of the Actor object's movement to the next cell in a path, as soon as the Actor object enters a cell, rather than when the center of a cell is reached. This caused a few unexpected problems where the path of the NPC would move diagonally and unexpectedly enter a side cell.

I'm not an expert on Navigation and Path-finding. This short introduction humbly tells me that it is a complex topic. Though: it gave me some good ideas as next-step to explore!
