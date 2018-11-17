---
layout: default
title: (Godot) Path-Finding
---
The first snow storm arrived today! Without having to commute, the first snow storm is quite pretty! It means we're cozying up inside, getting ready to hibernate for most of the next five or six months! Without outdoor distractions, I feel motivated to work on my game project! I think I finally have a plan that will give a decent game demo in a few months.

<h3>Path-Finding</h3>
All my cool work of the past month, on voxel and actor collisions was bound to have an impact on other sub-systems. When I introduced the wolf and tried spawning it at various locations in the world: I found that it did not always find a way to the village. It would occasionally get stuck under trees or staring at a cliff. Path-finding is a game system that absolutely must work. In any game, it gives me a feeling of unease when I see an enemy taking a longer path than what I perceive is obvious and available.

So, I gave myself a week to advance my path-finding technology. These are the issues that I had evaluated as must-fix:
1. All the path steps that are proposed must be possible by the AI agent.
2. The AI agent must jump only when necessary and jump reliably, so that it lands where the path wanted it to.
3. A path should include jumping-across a small cliff, instead of diving in and climbing out.
4. Slanted blocks are preferable to go up a level, rather than jumping.
Also, I wanted to add an automated test system to confirm that my improved code worked, as I changed it. That's what I spend my first day building and I don't regret it: I believe it saved me a lot of time!

<h4>Path-Finding through Collisions</h4>
I spent a lot of time brainstorming on my white board about the issue of how to achieve reliable path-finding in my current engine. The standard solution of using nav-meshes isn't great for modifiable voxel terrains. I tried really hard to come up with a decent logical solution to the problem, both using the voxel logically and generating non-axis-aligned boxes to collide with the world. These ideas all sounded complex, time-consuming to implement and to get right.

One evening, I came to the realization that the advancements in collisions and physics was causing these new issues with the path-finding. So, I decided to attempt using the collisions to find the paths. The solution comes with the path-finding taking very small steps and asking the physical world if the step is stable. A stable step is defined as a position where the actor does not collide with anything, and there is something solid underneath.

Hence, I take the world and sub-voxelize it into 5x5x5 path-finding steps per voxel. Each of these path-finding steps is an A* node. That's a lot of A* nodes! From each of these path-finding steps, I look for all the stable steps available, in the four standard directions (+X, -X, +Z, -Z). The function that finds stable steps for a path-finding point is where I can expand the path-finding to include more ideas, like jumping.

<img width="50%" src="../../../assets/path-finding-1.png"/>

<h4>Jumping</h4>
When computing the stable neighbor steps for a path-finding step: start with the four standard directions (+X, -X, +Z, -Z). For each direction, check the neighbor step and if that path-finding step is stable, then no more work is necessary: you can reliably walk in that direction. If it isn't stable, then the algorithm gets fancy and looks for walking up, walking down or jumping. If the path-finding point that is directly above or below the neighbor step is stable, then the actor can reliably walk up or down to this path-finding step.

When the neighbor step was unstable because it was filled with something solid, then I can check for a high jump by iteratively checking the stability of path-finding points above the obvious neighbor. If the obvious neighbor was unstable because it had no ground, then I might add up to two new path options: if there is a stable path-finding point below the obvious neighbor, then I can jump down to it. if there is a stable path-finding below further in the standard direction (within a range), then I can jump across to it.

<img width="50%" src="../../../assets/path-finding-2.png"/>

<h4>Path Smoothing / Compression</h4>
Right now, because the path steps are very restricted to the standard four directions, the AI agents do not move very naturally. I wish I had more time to work on path smoothing. Since my time is finite, I've decided to put this aside.

<h3>What's Next</h3>
I'm starting to get enough technology to build the base of my first game: Tabby Defense. I've added the Tabby as the heart of the game. I'm now working on the defenders in the next few weeks. More on this topic in a later blog!

<img width="50%" src="../../../assets/tabby-1.PNG"/>
