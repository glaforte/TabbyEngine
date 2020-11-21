---
layout: default
title: (Godot) Cooperative Gameplay
---
I enjoyed working outside quite a bit, this week! It started hot enough to move indoors and enjoy A/C in the afternoon. By the end of the week, I was working outside with a blanket on my knees. The temperature swing that we had this week is pretty crazy! I spent most of the week working on the cooperative gameplay module. I think I'm reaching an interesting level of completeness for the cooperative gameplay module. I also continued exploring some ideas in my voxel terrain engine.

<h3>Cooperative Gameplay</h3>
At the start of the week, I had implemented three different flavors of the application: <i>stand-alone</i>, <i>server</i>, <i>client</i>. I think a modern cooperative game must support joining in progress. This means that when the server starts, it feels identical to starting stand-alone. The goal is that each flavor will have its own local player character that the user can move around and its own local player inventory. Yet, all the players are a part of the same team, so they would share the base and its tasks.

<h4>Semantics</h4>
I've introduced a few semantics that I think are common enough in network software development. I find that establishing semantics is very important when describing my ideas and staying consistent takes discipline, but can really help in the long-run. When I write about the <i>server</i> and the <i>client</i>, they usually represent the processes. Godot also introduces the <i>master</i> and <i>slave</i> semantics. The <i>master</i> of a node is the one that controls everything that happens to it. When a node is accessed from a <i>slave</i>, it can read the node parameters, but if it wants to write to this node, it must marshal the command to the node's <i>master</i>.

When I write about <i>server user</i> and <i>client user</i>, I'm talking about the human behind the process. When I write about a <i>player</i>, I mean the set of nodes and visual cues exposed for the gameplay: the skinned character, the camera, the user interface, etc. There are many flavors of players: <i>local player</i> implies the player controlled by the current process, <i>remote player</i> means all the players not controlled by the <i>server</i> process.

<h4>Remote Players</h4>
My main struggle, this week, was to give the remote players a fluid gameplay experience. My first failed attempt was to marshal all the remote player actions to the server, to be executed. It was complex to set-up and required that I introduce new ideas, such as the server telling the client when its local player is ready to accept new actions. This idea failed because the remote players would have a very staggered experience. Even when running the server and the client on the same machine, the client user saw his local player actor and its camera stagger.

By mid-week, I was able to run one master and three clients together:

<img src="../../../assets/godot-networking-5.PNG"/>

I assumed that the erratic remote player movements was due to unordered deserialization in the Godot multiplayer API. When I worked on simulator software, we had fixed this issue by caching many frames and displaying them late but in order and synchronized on multiple machines/monitors. I wanted to avoid setting up this complexity, so I researched a different method to give the local player a fluid gameplay experience. I was halfway through an implementation of yielding ownership to the client processes for their local player, when I realized that the <a href="http://docs.godotengine.org/en/3.0/tutorials/networking/high_level_multiplayer.html">Godot high-level multiplayer tutorial</a> had a better solution. Godot supports setting a different <i>master</i> for each Node in the SceneTree. When the server spawns a remote player actor, it transfers its <i>master</i> ownership to that client. This allows for all player actions to be executed on the processes that own them. The server user sees the remote player characters move with staggering movements, but that's less important than the client users seeing their local player movements and animations properly.

<h3>Creative Module</h3>
This week, I continued advancing the voxel terrain. I had previously written an idea about adding voxels that are not solid cubes. The goal of the idea was to break the cubic-feeling that the user gets, when looking at the terrain. I rested on the idea for a few days and I decided to test it out when I needed a change from working on the cooperative gameplay module. I decided to name the shaped voxels: <i>terrain blocks</i>.

<h3>Terrain Block</h3>
The first terrain block that I supported was therefore the 1m^3 cube. The second terrain block that I introduced was a triangular prism that cuts a cube in half (a slanted block). So, a terrain block definition is its custom tessellation. Each terrain block has three parameters: a material, a shape and a rotation. I've used bit-flags to ensure that the terrain tiles don't require massive amounts of memory. I've introduced an option of <i>VOID</i> in the material parameter, but I think it might have fit in the shape parameter as well.

Before this idea, each terrain voxel would either hold a cube or an actor. With the introduction of the slanted block, it becomes possible for a terrain voxel to have one slanted cube and an actor sliding on top of it. I found it quite tricky to handle this case. Before this change, the actor had an implied volume of 1m^3. After this change, the actor rather has the remaining volume that's not taken by the terrain block. Giving the game actors a real control over their volume is definitely something that I need to handle in the coming weeks..!

Yesterday, I changed the terrain generation algorithm to introduce slanted blocks:

<img src="../../../assets/godot-voxels-8.PNG"/>

I think introducing the slanted blocks helped, but I'll need to introduce a few more block shapes!