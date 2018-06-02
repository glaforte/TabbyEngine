---
layout: default
title: Godot - Building Gameplay
---
It's been a beautiful sunny week here. After a few months of rest at home, I can honestly say it is hard to go to work. I've been unfocused on my Godot project, but I was able to introduce some new gameplay and AI improvements.

<h3>Building Gameplay</h3>
I've introduced the idea of characters doing building actions to construct huts, instead of poping them into existence!

<img width='100%' src="../../../assets/godot-building-1.PNG"/>

<h4>Village Team</h4>
I introduced a Team class in C++ class, extending the Node class. When the world is created, it creates a Player team and the Environment team. This idea comes from my other projects where the adversaries would look for the closest actor of another team, in order to attack them. Maybe some day, I will introduce a Wild team and add some wolves! In Godot, the Team class extends the Node class. The Team class is the parent to all the Actor objects that are part of the team. This has interesting properties for future development, like easily changing teams or splitting villages...

Right now, I'm using the Team class to hold a list of actions that need to be accomplished. I've changed the AI characters so that they check the Team's work-list and go to execute the next action in the work-list. If the work-list is empty: the AI characters still have the default action of selecting a random cell where they can do a short seed-planting animation. I've restricted the list of random cells to do the short seed-planting animation to a circle of 16m around the center of the world, so that the workers stay nearby, in case the player needs them!

<h4>Team Work-list</h4>
Sharing the work-list means that all workers and the player character can participate with the build action. The player now places the foundation of a hut; this foundation needs 20 build points to complete. When the build points have been attributed, the foundation actor replaces itself with the hut actor.

<h4>Recycler</h4>
This was the first time, in this project, that I had to remove actors. I introduced a new facade for this, similar to the Spawner C++ class, called the Recycler. Once I got the Recycler working for the foundation actor, I also added recycling of trees and stones once they are fully harvested by the player. Godot does spike when new Nodes are created, especially when instancing an advanced sub-scenes such as when spawning a worker. Having a Spawner and Recycler class should make adding Spawn Pools easier.

<h4>Dynamic User Interface</h4>
Using Godot's signal technology, I was able to introduce a simple dynamic UI. I'm especially happy about not overriding the "_process" function in my Godot scripts for UI elements! The "build_hut" button is enabled only if its input of two lumber items, is in the player's inventory. When the button is enabled and the player clicks it: the inventory is deducted right away and the dynamic UI reflects this. As well, the button will be refreshed, so that the button will be disabled if there is not enough lumber after the deduction.
