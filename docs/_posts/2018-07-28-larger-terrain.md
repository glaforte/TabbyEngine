---
layout: default
title: (Godot) Terrain Tiles II
---
Today is a nice and cool day. The heat wave and the rainstorms that followed it seem to finally be behind us. I've been struggling with some coding challenges at work: fixing issues related to really old versions of Qt. This has given me little motivation to code at home, this month, but I did progress on improving my terrain in my Godot project.

<h3>Terrain Tile Creation</h3>
A few weeks back, I introduced a debugging tool that let the user, with one click, force the creation of nine terrain tile. This week, I challenged myself to measure and to improve the time it takes to create each tile.

<h4>Fenced Height Values</h4>
This is the main idea that improved the performance of the terrain tile creation. By duplicating the perimeter height values from the surrounding tiles into a given tile, I trade-off memory and application complexity for a better performance. Since each tile holds 64x64 cells, the memory taken by the height values with one cell of fencing grew by 6% (66 * 66 / (64*64) - 1.0). This fencing does mean that any changes to the height values on one tile may need to be replicated to surrounding tiles, so that took careful coding to hide this new complexity away.

This idea increases performance because there are many algorithm that require the four vertices of a terrain cell. Two examples that related to terrain tile creation are: the creation of the terrain tile mesh that makes triangles out of those vertices; and the placement of the objects on  a tile that uses those vertices to find the center point of a cell. With fenced height values, these algorithms do not need to retrieve the height values for the perimeter cell's vertices from the neighboring tiles.

I did a lot of tests around the perimeter of tiles. I marked the perimeter cells with the yellowish material. I tested with both locking enabled and disabled. Here's a screenshot of a test showing that everything works well!

<img src="../../../assets/godot-larger-terrain-2.PNG"/>

<h4>Godot Child Nodes</h4>
Another source of performance loss was the addition of trees, rocks and rabbits in the 'Environment' sub-node of the Terrain. Turns out that when adding a child node to a parent node, in Godot: the engine will iterate over all the existing nodes in this parent node to check for duplicate names. This name-clash resolution functionality got very costly when the 'Environment' sub-node filled up with many children. This functionality does not seem to be removal from the Godot engine, so I introduced one 'Environment' sub-nodes per tile. Also, this discovery implies that you should not dynamically-add more than 40 or 50 children nodes to any parent node in Godot.

<h4>C++ Function Profiling in Godot</h4>
Surprisingly, I could not find any functionality to measure the time taken by a C++ function, in Godot. In 20 years of software development experience, I've commonly used a <a href="https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization">RAII</a> structure to measure the time taken by a specific code block or function. It makes for a great companion to a sampling profiler. I researched a cross-platform alternative to the QueryPerformanceCounter and QueryPerformanceFrequency functions of the Windows SDK. I was happily surprised to see that C++11 wrapped the functionality. Here's the code:

<pre>
class PerformanceMeasureBlock {
public:
    PerformanceMeasureBlock(const char *name) :
            block_name(name),
            start(std::chrono::high_resolution_clock::now()) {
    }

    ~PerformanceMeasureBlock() {
        std::chrono::high_resolution_clock::duration duration = std::chrono::high_resolution_clock::now() - start;
        std::stringstream output_string;
        output_string &lt;&lt; "[" &lt;&lt; block_name &lt;&lt; "] " &lt;&lt; (static_cast&lt;double&gt;(duration.count()) / 1.0e6) &lt;&lt; "ms" &lt;&lt; std::endl;
        OS::get_singleton()->print(output_string.str().c_str());
    }

    std::string block_name;
    std::chrono::high_resolution_clock::time_point start;
};
...
void function_to_measure(void) {
    PerformanceMeasureBlock block("function_to_measure");
    ...
}
</pre>

<h4>Results</h4>
I used the newly-coded PerformanceMeasureBlock structure and the tile-creation debug tool to measure the time used to create a set of nine tiles, before writing any optimization code. I used the same technique at various steps when optimizing, to ensure that my changes were indeed helping. Before changing the code, the measurements were 4500ms for the first set of tiles, 5300ms for the second set of tiles and increased by about 800ms for each new set of tiles. After all my changes, the measurements were 1800ms for each new set of tiles, without growing. That's only 200ms for each new tile, so I'm satisfied for now!

