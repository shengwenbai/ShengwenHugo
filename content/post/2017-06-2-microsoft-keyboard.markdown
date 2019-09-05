---
layout:     post
title:      "How to: use the zoom slider to scroll on Microsoft Natural Ergonomic 4000 keyboard"
subtitle:   "Ergonomic keyboard save your wrist"
date:       2017-06-02 12:00:00
author:     "Shengwen"
header-img: "img/Keyboard.jpg"
tags:
    - Stuff
---

I've been using Filco Majestouch as my coding keyboard for 2 years and was satisfied with it. But recently I've been suffering from pain in my left wrist and I realized maybe it's time to get an ergonomic keyboard. Finally Microsoft Natural Ergonomic 4000 is what I got. Though the typing experience is not good as a mechanical keyboard, it does save my left wrist. 

There're a lot of useful functional buttons on this keyboard, including a zoom slider in the middle. A zoom slider may helps a graphic designer working with Photoshop, but it seems to be useless for a programmer. Here is the way to use it for scrolling instead.

1. Download and install Microsoft Mouse and Keyboard center.
2. Modify the command.xml file. (By default, the location is C:\Program Files\Microsoft Mouse and Keyboard Center). Find and replace all <C319 Type="6" Activator="ZoomIn" /> as <C319 Type="6" Activator="ScrollUp" />, and all <C320 Type="6" Activator="ZoomOut" /> with <C320 Type="6" Activator="ScrollDown" />
3. If you don't have the authority to modify the xml file. A Simple way is to copy it to desktop, modify the copy and replace the original file. Backup recommended. 