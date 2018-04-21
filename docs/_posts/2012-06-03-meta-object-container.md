---
layout: default
title: Meta-Object Container
---
<p>We started using Qt, at Feeling Software, in February 2009. I present
 to you two classes that we added to the framework very early on: the 
Object Container and the Meta-object Dictionary. These are at the core 
of my current personal project. While personal projects tend to come and
 go, core classes such as these tend to stay for a long time.</p>
<h2>Object Container</h2>
<p>The Object Container is a simple pattern that I coded for the first 
time in 2006, after being burned by reference-counted pointers. It 
reverses the idea of the having your memory reference counted, by 
instead tracking all memory allocations. Every memory allocation may be 
tracked multiple times, but it is contained only once. The container is 
responsible for releasing the allocated memory. Right before releasing 
an allocated block of memory, the code must inform all trackers that the
 allocated memory is about to dangle.</p>
<p>Qt already implements this simple idea. The QPointer template and the
 QObject's parenting tree make the framework awesome. But it is not 
complete: it is sorely missing a list of tracked pointers. You could use
 a QList&lt;QPointer&gt;, but you'd be sad when encountering NULL values
 in your list. At the core of the Qt framework, there is also the 
signal/slot system. When you put the two ideas together and apply some 
magic, you get the very powerful Object Container list template.</p>
<p>The Object Container list template tracks memory allocations and 
shrinks when those allocations are released. More importantly: it emits 
signals when objects are added to it, and when objects are released from
 it. The implementation of the Object Container list template that I use
 currently is mostly written by Alexandre Cossette-Pacheco.</p>
<h2>Meta-object Dictionary</h2>
<p>The Meta-object Dictionary is self-explanatory. I was originally 
surprised about the lack of such a structure in the Qt framework. A 
complete reflection of the code is impossible without it. Having a full 
dictionary of the meta-objects in your code has many applications, such 
as:</p>
<h3>UI Plug-ins</h3>
<p>You can request the list of UI classes that implement a given 
interface from the Meta-Object Dictionary and instantiate them. This 
works well in tabbed panes, where each tab is a separate UI class. The 
main issue that we found, at Feeling Software, with this approach, is 
ordering: it is impossible to effectively control the order of the tabs.</p>
<h3>QVariant and QObject*</h3>
<p>There is a flaw in Qt's QVariant class. If I declare a meta-type for 
the QWidget* type and create a QVariant with the type-id of the 
QWidget*, then doing qwidget_variant.value&lt;QObject*&gt;() always 
return null, because the integer type-ids don't match. The QVariant is 
unable to down-cast pointer values.</p>
<p>It is even worse in this scenario: I write a nice class using the 
QVariant, expecting a QWidget* inside and publish it to developers. 
Someone else writes a application, wanting to use my class. Instead of a
 QVariant containing a QWidget*, he provide an instance of my class with
 a QVariant that contains a QMainWindow*. Even though the two QVariant 
types should be compatible, my class will fail.</p>
<p>The Meta-Object Dictionary can solve these two problems by tracking 
the registered QVariant type-id of all the classes in your code. You can
 now confirm, with the Meta-Object Dictionary, that a QVariant object 
holds a pointer to a QObject-derived class and grab the QVariant's 
internal structure in order to reinterpret_cast it to QObject*. Then, 
you can safely attempt to up-cast the QObject pointer into your own 
class.</p>
<p>One draw-back of my implementation that tracks all these type-ids, is
 that some of the QVariant type-ids may never be used: which may be 
polluting the QVariant type-id system. I've never measured how much of a
 problem this might be.</p>
<h3>Binary Chunk Serialization</h3>
<p>This is what I'm currently researching. Instead of serializing out 
using hard-coded chunk headers, I write out the 32-bit hash value of the
 namespaced class name. When serializing back: you can find the 
meta-object from the dictionary and invoke the appropriate constructor. 
My favorite part of this is that the serializers can be very low-level 
in the architecture and do not need to be modified when additional 
serializable classes are added to the project. The main draw-back is the
 serialization function belongs to the serializable class. I wish I 
could move the serialization function outside of the data class, and 
into some serializer in order to trim the data class.</p>
