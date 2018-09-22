---
layout: default
title: Godot - Player Character II
---
This week has been very challenging, technically. I started the week smoothly by linking the creative module with the inventory, and adding more block materials. Then, I decided to tackle camera controls and player movements. Last week, I was able to make a simple structure, but my old camera controls made it impossible to build ceilings. Also, it was too easy for the camera and the player mesh to be placed inside a block. I've made some nice progress to reduce these issues and that's the topic for this week!

<h3>Character Mesh</h3>
I've been using Blender in the weekends to improve the character mesh. I've finally moved away from multiple mesh pieces and switch to using a skinned mesh. The results are dramatically better. I find that modeling in Blender is getting easier as I attempt more advanced challenges. And modeling a human being is definitely an advanced challenge!

Here's a before and after shot. What do you think?

<table>
      <tr>
        <td><b>New Skinned Mesh</b></td>
        <td><b>Original Multi-Mesh</b></td>
      </tr>
      <tr>
          <td><img width='100%' src="../../../assets/blender-character-skin-4.PNG"/></td>
          <td><img width='100%' src="../../../assets/blender-character-original-1.png"/></td>
      </tr>
</table>

<h3>Character Movement</h3>

<h4>Box Volume</h4>
I have struggled quite a bit in the past month with the player movement. I knew that I would eventually have to give volume to the character mesh; so far, my mathematics had been assuming that the character was a particle in 3D. Researching on the Internet, I found that AABBs are the easiest way to tackle this problem.

<img src="../../../assets/godot-character-1.png"/>

A quick glance at Godot's documentation and I confirmed that its AABB class and its Face3 classes have what I need: the triangle-AABB intersection function. Thanks Godot :) I use a smallest AABB than the real AABB of the character mesh because it gives better results when the character is on a cliff: you don't want to see the character's feet as hovering beyond the edge of the cliff.

The other side of the problem: my current character movement code uses ray-casting that outputs the intersection point and the intersection normal. The intersection point is used to advance the position of the character and the intersection normal is used to slide the character when it hits a wall or ceiling. I tried various ways to emulate the intersection normal from the Triangle-AABB intersection, without success. One evening, while brushing the cat, I had a decent idea: I should change the character movement algorithm instead of fighting with the Triangle-AABB intersection code to output some overlap minimum translation vector. A few minutes of brainstorming later: I decided to split the movement into four parts that are evaluated independently.

With this idea, the box-casting function can take a box and a movement vector and return the furthest position that the box can take before hitting the terrain or other characters. The character movement function hence does four box-casts.

<img src="../../../assets/godot-character-2.PNG"/>

If the character is on the ground, a first box-cast is done with a small step in positive Y. This is necessary to support climbing up a sloped terrain block. The second and third box-casts move the character sideways: using the X and Z components of the movement vector, separately. This gives the feeling of sliding when hitting a wall. Since the world is mostly cubes, that works well! Finally, the fourth box-cast uses the Y component of the movement vector. If the player was on the ground, it adds a small step-down. If this box-cast hits something and the Y component of the movement vector is negative, then the character is assumed to be on the ground for the next iteration.

My first successful attempt for the box-casting function moved the box in steps of a centimeter. This had the side-effect of the character shaking up and down when standing idle. The box-cast has to be very precise to remove all this shaking. I tested the algorithm with a step of a milimeter and that was better but very expensive when there were many characters in the scene. To fix the performance and improve the quality of the result, I changed the box-cast algorithm to be similar to a binary search.

The binary-search version of the box-cast first computes if the box overlaps the world in its start and final positions. If the box does not overlap the world in its final position: that's where the character moves. I realize that this could lead to the character teleporting through objects, but for now: the movements are small enough and the character is slow enough that I'm confident this won't happen. If the box overlaps the world in its start position, the algorithm stops there: that means a block suddenly appeared in the character's volume. The algorithm, at this point, has a valid start position and an invalid end position. It takes the half-position between the start and final positions. It evaluates whether the half-position overlaps the world. If the half-position overlaps the world: the algorithm computes the 1/4 position; if it does not overlap the world: the algorithm computes the 3/4 position. And so forth, until the distance between the current position and the next position is less than 1 milimeter. The box-cast then returns the furthest valid position found.
