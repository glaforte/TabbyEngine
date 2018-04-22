---
layout: default
title: QML Garbage Collection
---
<p>I was investigating a strange destruction of my UI elements, yesterday. The UI that QML was automatically generating would simply stop appearing. That UI is used to modify the more complex rendering objects. I added a console output statement inside the JS loop that is populating the UI frame. After a few minutes of using the UI: it showed that the upgraded meta-object dictionary stopped to provide all the values that I had pushed into it, at the start of the application. These values are not used in the C++ code, after being pushed into the meta-object dictionary and they are meant to be accessible through the whole application lifetime.</p>

<p>That obviously screams at the problem being with the QML garbage collector, so I researched the way the QML garbage collector works and found this page about the QDeclarativeEngine and object ownership. The idea of object ownership is simple: each object provided to QML can either be owned by QML or owned by the underlying C++. Any object owned by QML may be garbage collected when memory runs low. Any object owned by C++ is outside the influence of the garbage collector.</p>

<p>So far, so good. What I'm peeved about is the heuristic that sets the default ownership of objects:</p>
<div class='code'>Generally an application doesn't need to set an object's ownership explicitly. QML uses a
heuristic to set the default object ownership. By default, an object that is created by QML
has JavaScriptOwnership. The exception to this are the root objects created by calling
QDeclarativeCompnent::create() or QDeclarativeComponent::beginCreate() which have
CppOwnership by default. The ownership of these root-level objects is considered to have
been transferred to the C++ caller.<br>
<br>
Objects not-created by QML have CppOwnership by default. The exception to this is objects
returned from a C++ method call. The ownership of these objects is passed to JavaScript.</div>

<p>The <a href='http://doc.qt.io/qt-5/qqmlengine.html'>above exerpt</a> is from the official Qt documentation. The exception stated in the last line really surprised me. I cannot imagine how inefficient it would be if every C++ function called from some JS function had to return primitive types or newly-allocated objects. I believe that in order to keep symmetry, a JS function that requests a new C++ object should find the correct place to release it. Here's an example:<p>
<pre><code>function worker(objectClassName) {
    var object = BMetaObject.construct(objectClassName);
    object.process();
    object.deleteLater();
}</code></pre>

<p>For now, I change the object ownership change my JS loop so that the garbage collector doesn't wipe my persistent objects:</p>
<pre><code>var editableProperties = BMetaObject.classProperty(className, "editableProperties");
BMetaObject.setObjectOwnership(editableProperties, 0);
if (editableProperties != lastValue) {
  typeEditorModel.insert(0, {"className":localizedName, "customEditor":null,
                                       "editableProperties":editableProperties});
  lastValue = editableProperties;
}</code></pre>

<p>...<br/>
Henceforth, I will remain mindful of the object ownership in QML! Hopefully, you'll avoid this pitfall.