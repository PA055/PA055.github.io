---
layout: post
title: "GUI's \U0001F62D"
description: A blog post about my experience creating an LVGL GUI for my robotics team
category:
- Robotics
tags:
- GUI
- 5839B
- LVGL
---
The robotics reason recently started and with out first competition comming up in a couple of months, i need to start working on the code for our robot. I'm using an open source library I contribte to for this called [LemLib](https://github.com/LemLib/LemLib) which provides a powerful movement system for our robot. However this library uses multiple [PID controllers](https://en.wikipedia.org/wiki/Proportional–integral–derivative_controller), which are hard to tune precicly without a method of graphical feedback. So thats my main reason for creating a GUI, to create a graphing screen that gives me straightforward feedback and allows me to quickly tune the PID controllers. As you can tell by the title, this is causing me a lot of pain.  

Ok so what exactly do i need in this? Since im going to need to tune an unknown amount of PID's i would like to be able to easily add a PID and it just work. I also need to tune 4 different parameters for each PID: the P, I and D terms and also the slew for drivetrains.
