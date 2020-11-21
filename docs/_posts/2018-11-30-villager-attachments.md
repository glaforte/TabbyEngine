---
layout: default
title: (Tabby) Villagers
---
There's only one month left in 2018 and I'm looking forward to turning the page. Working on my projects definitely gives me the feeling that I'm back on the right track! Also, I started contributing to the Godot project, two weeks ago. I've been fixing and advancing the reported issues related to the COLLADA import/content pipeline, since I do have an expertise with this file format. I can feel that juggling both the open-source project contributions and my own project advances will be tricky, but I enjoy contributing back to the Godot community.

<h3>Villagers</h3>
I've progressed on two key elements, in the past two weeks. The game time has now been split into three periods that create a circadian cycle: 
* The day, when construction and gathering happens;
* The dusk, when predators attack;
* The night, when villagers rests and the game saves.

The players can now change which villager they control, giving them a better control over their placement at dusk.

<table>
      <tr>
        <td>Approaching Dusk</td>
        <td>Dusk</td>
      </tr>
      <tr>
          <td><img width='93%' src="../../../assets/ApproachingDusk.png"/></td>
          <td><img width='100%' src="../../../assets/Dusk.PNG"/></td>
      </tr>
</table>

<h4>Bone Attachment</h4>

In order to improve the variety of the villagers, I've added a few hair-styles. Trying to get the hair-styles to attach to the 'Head' bone in Blender and in Godot proved easy, but gave different-looking results. I researched whether anyone had posted any notes about how to inter-operate bone attachments between Blender and Godot, without success.

Digging into the skeletal animation systems of Blender and Godot, I managed to explain the difference: In Godot, a bone has a position, an orientation and a parent. In Blender, a bone has a start position and an end position, a roll angle and a parent.

This distinction means that in Godot, when attaching an object to a bone, its transform will take in the bone's position and orientation. This is what's done in the graphics engines that I've used in the past. In Blender, though, the attached object's transform will use the end position of the bone as the transform origin, along with the bone's orientation. I experimented with attaching to the parent bone in Blender and the actual bone in Godot, without success. That gives me the expected position. But the orientation is wrong then. There is just no option in Blender to attach to the bone's start position.

The working idea that I came up with is simple: attachable bones should be very very short. If you are experiencing a similar issue, here's a couple of rules to follow within Blender to make sure that you can attach meshes to a bone in Blender and in Godot and get similar results.
1. For a bone that has no children, the easy solution is to shorten the bone down to one milimeter or less in length. Since the dimension of my game is in the meters, the one milimeter excess in Blender does not impact the look significantly.
2. For bones that have children, you have no choice but to add a new tiny bone. For example, if you want to add backpacks in your Godot game, you should add a really short 'Back-Attachment' bone that protudes from the 'Torso' bone.

<h3>Variety</h3>

As a result, I've modeled a couple of easy hair-styles; I now have a system where I can predictably make new mesh-accessories for the villagers.

<img width="100%" src="../../../assets/hair-styles.PNG"/>

<p>I've added a woman version of the villager body. I added 6 possible hair colors. Since a bald hair-style doesn't use the hair color, that gives 13 hair combinations. Along with the clothing styles, clothing colors and skin colors that I had prototyped in the past, that gives me 1170 possible combinations for villagers. That's still just a start, but I'm happy to say that I have yet to see duplicated in my 8 randomly-generated villagers, since adding these options!

