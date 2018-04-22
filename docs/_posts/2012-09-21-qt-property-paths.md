---
layout: default
title: Qt Property Paths
---
All half-decent 3D scene editors need an object property browser. I've discussed using QML and meta-programming concepts to simplify the coding of a property browser before, in this post. More recently, I've upgraded my property browser with a new idea: showing the properties of sub-objects. While coding that: I managed to simplify the QML code by re-hashing an my earlier adaptation of QML property paths.

<h3>Qt Relative Property Path</h3>
In the TEngine, most objects contain a few primitive values and zero or more references to other assets. For example, when adding a light to a scene, a TNode is created that references the user-selected TLight. The purpose of the TNode is to give a 3D placement to the TLight. For this purpose, the TNode exposes the following properties:
<pre><code>Q_PROPERTY(BVector3F scale READ scale WRITE setScale NOTIFY scaleChanged)
Q_PROPERTY(BVector3F rotation READ rotation WRITE setRotation NOTIFY rotationChanged)
Q_PROPERTY(BVector3F translation READ translation WRITE setTranslation NOTIFY translationChanged)
Q_PROPERTY(BAsset* instance READ instance WRITE setInstance NOTIFY instanceChanged)</code></pre>
This is a simplified version of the TLight:
<pre><code>Q_PROPERTY(BVector3F color READ color WRITE setColor NOTIFY colorChanged)
Q_PROPERTY(float intensity READ intensity WRITE setIntensity NOTIFY intensityChanged)</code></pre>

<p>A property path is a string that represent a traversal through multiple objects, starting from a provided object. For example, when starting from a TNode object; "instance.intensity" can be used to access the intensity of the light currently instanced by the provided node.</p>

<p>I came up with the idea of introducing property paths in the C++, similar to the ones in QML, earlier in the year, while working on Morpher, a predecessor to Krash. The problem that I was trying to solve, was to automatically connect signals and slots for objects deeper than the current level, before the intermediate objects exist. For example, in the constructor of a TNode: I would create a DeepConnector with the following constructor:</p>

<pre><code>DeepConnect(this, SIGNAL(instance.intensityChanged(), this, SLOT(doSomeWork()));</code></pre>

<p>The DeepConnector connects to the "instanceChanged" signal and when the instance is valid, connect to its "intensityChanged" signal. When the "intensityChanged" signal is raised, the DeepConnector calls "doSomeWork". Sound simple enough? I implemented the DeepConnector, simplified my application code and put the idea aside.</p>

<h3>Qt Absolute Property Path</h3>
<p>Last week, this is the problem that I tackled: I was adding to my property browser, a button to expand, in-line, an object referenced by one property in the property browser. That's a feature that I have seen in other property browsers and I thought it would be great to include it in mine. Implementing this feature is simple enough and my first implementation included a large amount of boilerplate code. I hate boilerplate code, so I looked for a better way to implement a property browser tree, using property paths.</p>
<p>That's when I came up with absolute property paths. All serializable assets in the TEngine can be referenced by a unique string, the asset reference. By tagging the property tag at the end of an asset reference, I can identify all my asset's properties without providing my user interface code with any objects. I've already ranted about the QML garbage collection, so you can review that blog post if you want to know why providing objects to QML isn't awesome.</p>
<p>For example, when opening a light node in the property browser, the QML code for the intensity field edit is provided with the absolute property path: "Scene1.package#Node1.instance.intensity". The QML code pushes the absolute property path to a local structure that it can use to retrieve/modify the current intensity value and track the intensityChanged signal.</p>

<img src='../../../assets/property_browser.jpg'/>

<h4>Object List Support</h4>
Most object references in the TEngine are admittedly not as simple as the TNode instance. Most object references are contained in lists. For example, the TMaterial contains a list of TTextures. Expanding the idea of property paths to support object lists requires a minimum amount of straight-forward code. This is an example of a absolute property path going through an object list: "StandardMaterials.package#Wood.textures[0].offset.x". Treating the index of an object list item as any other object property works well enough.

<h4>Undo/Redo Functionality</h4>
Absolute property paths can also be used to greatly simplify the interface between the property browser and Qt undo stack. First, I changed my property browser is do all its work using the QVariant structure, as described in this documentation page by Nokia. Recording a change done by the user, in the property browser, boils down to recording two QVariant structures and one absolute property path. Oh, and while on the topic: lambdas are awesome for undo/redo.

<pre><code>void MPropertySupervisor::setPropertyValue(QString absolutePropertyPath, QVariant newValue)
{
   QVariant oldValue = BPropertyPath::value(absolutePropertyPath);
   executeUndoableCommand(
          [=] () { BPropertyPath::setValue(absolutePropertyPath, newValue); },  /* Undo */
          [=] () { BPropertyPath::setValue(absolutePropertyPath, oldValue); } /* Redo */
   );
}</code></pre>

<h4>Proxy Serialization</h4>
As mentioned in this release note, I've been working on implementing a generic object proxy system. Recording the state of a proxy object is very similar to recording an undo stack. So, property paths come in handy when serializing a proxy: I serialize the reference to the original object as well as all the modifications done to it using one relative property path and the modified value as a QVariant.
