---
layout: default
title: Godot - Inventory
---
I'm happy to say that I'm once again, gainfully employed. The project that I will be working is very interesting and is not related to graphics nor video games. This will reduce the time that I have to play around with Godot and other technologies. I will keep this blog alive, showing off small improvements and ideas. I spent the last week trying out the UI facilities in Godot as well as the database concepts that it offers.

<h3>Inventory</h3>
In my personal project, as of today, when you press the 'I' key, you have shown the following dialog:

<img width='100%' src="../../../assets/godot-inventory-1.PNG"/>

Working with the UI elements in Godot was straight-forward. With a few mouse clicks and no reading of documentation, I had a working split-window in a transparent, modeless dialog!

<h4>Modality</h4>
Godot's input map is very global, while a UI-dedicated technology like Qt propagates input events through the tree of widgets. This means that introducing modality of the Inventory dialog was tricky. I took me a first hours of meditation to figure out how to properly introduce modality. I decided to introduce a LocalPlayer singleton class that would restrict inputs. Maybe the LocalPlayer singleton class could also be useful if I want to experiment with co-operative gameplay, later?

<h3>Database</h3>
A few weeks ago, I did some challenges in SQL on <a href="https://www.hackerrank.com">HackerRank.com</a>. A few of the companies that I was in discussion with, had mentionned that SQL was important to them. As well as some SQL query challenges, I reviewed the basics of relational databases, primary keys, foreign keys, natural keys...

When I coded the first version of the Inventory class, I used strings to identify what type of stuff was contained in the Inventory. I knew, from the start, that a string was not expressive enough to make an interesting UI. Last week, I introduced the Commodity class to describe each stuff that could be contained in an inventory. Then, I promptly added a CommodityDatabase singleton class to contain all the possible commodities. I kept the string as a key, so the Inventory class remained intact. In order for the player to build a hut, I introduced the ConstructionProject class and the Building class. A ConstructionProject object uses commodities as inputs and a Building object as output. I wrote a first draft of the ConstructionProjectDatabase singleton class and the BuildingDatabase singleton class before I scrapped them. I did not want to have to code two new classes everytime that I introduced a new type of possible data.

That's when I realized that I was designing a relational database. I hesistated using the <a href="https://github.com/khairul169/gdsqlite">SQLite wrapper for Godot</a>. I think using SQLite would eventually have made writing save-games easier, but its drawback is that extending the Database would have been harder. Also, I won't pretend that my personal projects are complex enough to need the memory paging abilities of a SQL database engine. I settled to use the JSON parsing built-in to the Godot engine. As with a lot of building blocks in the Godot engine, it was surprisingly pleasant to use. In about 20 lines of C++ code, my project iterates over all the .json files of a specific folder, reads them and adds their data to the Database singleton class.
