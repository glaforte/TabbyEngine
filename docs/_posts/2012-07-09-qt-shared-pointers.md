---
layout: default
title: (QT) Shared Pointers
---
As a part of the Tabby Engine project, I have been reviewing how to do multi-threaded rendering. In my design: I've been feeling the need for a simpler memory tracking technique.

For the purpose of this post: one of the structures that I would like to share ownership across multiple threads is the TTexture. This class is based on top of QObject and a few structures may hold references to it: TMaterial, BPackage and HDrawItem. BPackage represents one file containing assets, such as textures; the BPackage objects are exclusive to the main thread: where the application may decide to load or unload a package at any time. Unloading a BPackage is very fast and may be triggered right after pushing a frame to the renderer. HDrawItem is the small structure sent to the rendering thread which holds the minimal amount of information required to draw one frame.

Shared Pointers are commonly used for this reason, so I decided to check out Qt's shared data and shared pointer structures. There is a lot of documentation on Qt's many shared data/pointers structures, so I will not attempt to describe them. If you are not familiar with them, I suggest reading <a href='http://blog.qt.io/blog/2009/08/25/count-with-me-how-many-smart-pointer-classes-does-qt-have/'>this very informative QtLab blog entry</a> by Thiago Macieira.

There are two competing implementations in Qt that I checked out. After further reading about QSharedData and QSharedDataPointer, I did not attempt to prototype it. Although the implementation details are very close to classical reference counting that I'm used to: I'm very scared by the idea of my texture objects being automatically 'detached'.

<h3>QSharedPointer and QWeakPointer</h3>

It took me a couple of readings to understand the idea behind the QSharedPointer/QWeakPointer classes. In a couple of sentences: Qt will create a small object to wrap the provided pointer and keeps the reference count of the provided pointer in that small object; this creates two levels of indirections. The shared pointers reference the small object which references the original pointer. Using the QSharedPointer and QWeakPointer everywhere should, in theory, prevent dangling pointers completely - a very worthy endeavor, so I started prototyping using QSharedPointer and QWeakPointer is my project.

I found that this simple and innovative structure becomes problematic if your program uses QML extensively. I lose track of that small object when I provide a TTexture pointer to QML. I cannot provide a QSharedPointer<TTexture> to QML because I routinely cast to BAsset&#42; or QObject&#42; inside the QML declarations. When a C++ function receives back a TTexture&#42;, it cannot increase the reference count of the texture directly: it must go back to the package or some other structure with the QSharedPointer<TTexture> and search for the TTexture, which is inefficient.

I found a second smaller drawback of the QSharedPointer system is that I would have to register each object type thrice as QVariant types: TTexture&#42;, QWeakPointer<TTexture> and QSharedPointer<TTexture>. This registration is required in my project because of the MetaObjectDictionary, which is central in the architecture and is explained in a previous post.
After this prototype, I wrote my own classical invasive shared pointer template, using Qt's QAtomicInt. A word of caution: shared pointers of any kind, don't play nice with Qt parenting of objects. All reference-counted objects deriving from QObject should have NULL parents.

<h3>Shared Pointer Lists</h3>

Once I had implemented the counted reference template: I looked at how to deal with lists of reference counted objects. I quickly realized that I needed to duplicate the object list template. I would recommend to anyone using shared pointers to avoid duplicating the code of your object lists or object containers.

In some cases, you may want the list of reference counted object to increase the number of references: for example, the package should contain all the assets until it is unloaded. In some cases, you may want the list of reference counted objects to NOT increase the number of references and tracked the pointer, so that it can be nulled or removed if the reference counted object is released: for example, the material editor UI will track the list of textures but should not hold back the release of the texture objects.

Once I realized how much change that would require and how annoying it would be in the long-term, I looked for a simpler alternative. I recommend that instead of duplicating the template code: check for a reference counted meta-type when inserting/removing objects in the object container. I think it is worth taking the small performance hit in order to simplify the code. Making the developer's life better is a worthy endeavor too!
