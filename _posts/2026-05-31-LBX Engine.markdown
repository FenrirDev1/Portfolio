---
layout: post
title: LBX Engine Input
permalink: /LBX
yamlName: LBX
video: false
picture: true
show: true
---


### Abstract

LBX engine was the last module in University of Staffordshire with the task of working as a group to create a cross platform game engine. In which I worked on the Input system.

The input system I created was platform agnostic 'managers', and platform specific 'handlers'

Managers have the functions like *button pressed* for their device (keyboard, mouse, controller etc.).

Handlers are platform specific, so PC (*GLFW*) and Playstation 5, these send device info to the managers so they can complete their queries.

### Why?
---

This system was built in mind to be expandable quickly to more platforms whilst also being abstracted easily at the user level, since managers don't contain platform data, they just don't update their info meaning inputs checks for inapplicable devices just return false.

It also means devices across platforms, i.e. controllers, can be written regardless of platform, and when switching platforms it automatically changes to the other platforms handler and still allows for same input queries.

### Reflection
---

The biggest takeaways from this project was the team work and group work, working with different parts of the engine required constant communication. The engine took a number of big rewrites and keeping up to date and changing my system to still work needed sit downs with the team and working through any issues that arose.




