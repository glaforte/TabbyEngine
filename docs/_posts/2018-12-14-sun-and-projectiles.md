---
layout: default
title: (Tabby) Projectiles and Performance Spikes
---
The flu struck me, early this week. I started the road to recovery, today, but I have to admit: I worked maybe eight hours in the past three days. The quality of this blog post is probably being affected by my coughing!
I've tried to keep the topic of hunting and predators alive during this short bout of sickness, but the best upgrade to TabbyDefense were done in that first week.

<h3>Projectiles</h3>
I took multiple steps to implement projectiles. All defense games require projectiles, correct? In a 3D world with a tribal feel, that means spears and arrows! With this idea in mind, I started that first week throwing rocks. The physical movement code that I wrote months ago is in a good shape, so it was pretty straight-forward to add a keystroke that emitted a 'rock' entity with a starter velocity. I did have to add some callback scripts functions when the physical actor collides with the terrain or another actor. Upgrading the throwing of rocks to throwing spears meant not only adding a new animation, but also how to aim the spear to somewhat reliably it? That's where <a href="https://en.wikipedia.org/wiki/Projectile_motion">Projectile Motion</a> formulas come in.

<img width="70%" src="../../../assets/projectile-2.gif"/>

<h3>Performance Spikes</h3>
Last week, I also decided to start addressing those performance spikes that are making the game hard to test, even for its creator.

<h4>Sun Movement</h4>
By far the most pronounced performance spike was when changing the angle of the sun in the <a href="https://docs.godotengine.org/en/latest/classes/class_proceduralsky.html">Procedural Sky</a> within the Godot Environment.
I had previously prototyped the <a href="https://github.com/BastiaanOlij/godot-sky-asset">godot-sky-asset</a> project by Bastiaan Olij.
The godot-sky-asset project uses a shader to generate the sky on the GPU, so that is much preferable to the CPU computations done by the standard Godot Procedural Sky.
Since I work on the development branch of Godot, it took two failed iterations before properly integrating the godot-sky-asset, but now it looks gorgeous and is blazing fast!

<img width="70%" src="../../../assets/sunrise-1.PNG"/>

<h4>Thread Pool</h4>
I do profiling of the frame-time of this project very regularly, at least once a week, so I knew about more spikes and I knew that to fix them, there were two options: multi-threading or simplifying features.
So, I decided to write a simple C++ thread pool for Godot. The idea is that it would fit perfectly within Godot by accepting its variants structures.

<pre>
class WorkItem : public Reference {
	GDCLASS(WorkItem, Reference);

	virtual void do_work() {}
};
</pre>

<pre>
class WorkerThreadPool : public Object {
	GDCLASS(WorkerThreadPool, Object);
	static const int worker_thread_count = 4;
...
public:
	static WorkerThreadPool *get_singleton();
	void add_work_item(const Ref&lt;WorkItem&gt; &work_item, const NodePath &reply_node, const String &reply_function_name);
	void poll();
	void flush();
...
};
</pre>

The WorkItem class is built on the Reference class so that it can easily go in a Variant. This class is meant to be overloaded, but I'm toying with the idea of using lambdas or having a scripted work-item.
The WorkerThreadPool is a singleton that spawns four threads right from the start. It also contains a few synchronization structures and two queues: the received work and the completed work. The 'add_work_item' function adds to the received work queue. One of the child thread is awaken when a new work item is received using a semaphore. The child thread executes the work item and places it in the completed work queue. Once a frame, the main thread calls the 'poll' function. This function clears the completed work items, calling each work item's 'reply'. The 'reply' function call is recorded as a node-path and function-name pair, to avoid dangling pointers. The signature of the 'reply' function class should have exactly one argument: the work-item reference is provided back.

<h4>Voxel Tile Mesh Creation</h4>
The first operation that was transformed into a work item. In all my profiling, voxel tile mesh creation was consistently in the top 3 problem areas. Since I've moved trees and rocks into the voxel grid, it dirties more often and now: AI agents harvesting are causing the voxel grid to be dirty - not just the player's actions. I encountered a few issues here, mostly with Godot mesh and materials. While Godot claims to be thread-safe when handling its resources, I think the bugs that I encountered prove that assertion wrong. My conclusion is that very few operations in Godot can be done outside the main thread; hence, the work items that I design should push to a separate thread only the expensive code that I have ownership over.

<h4>Voxel Tile Load, Generation and Unload</h4>
The second operation that was transformed into a work item. Before my work, the heuristic to check whether to load or unload a tile already triggers on an arbritrary 1-second timer, so pushing the operation into a work-item that is recovered a few frames later causes no problem with timing during gameplay. For the game creation, I did add a new function to the thread pool that 'flushes' it: it waits until all the queued operations are done and triggers their replies.

<h4>More Performance Spikes</h4>
I've identified a few more operations that would benefit from using work-items, in order of diminishing returns.
First and foremost is path-finding: delaying the movement of AI agents for one or two frames, while their paths are being computed is probably acceptable.
Second is the auto-save process: the user would appreciate if the night is short!
