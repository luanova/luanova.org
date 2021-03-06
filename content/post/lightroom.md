+++
title = "Lua: how we first met"
date = "2007-11-04"
description = "Lua's use in Adobe Lightroom inspired a look."
author = "nathany"
+++

This past summer I evaluated both Apple Aperture and Adobe Lightroom for managing my digital photographs. You see, iPhoto 6 was getting a bit sluggish, Photoshop Elements even more so (not yet having Intel support). To top it off, I was considering a Digital SLR, so I'd need something that could read RAW.

Adobe Lightroom won out for me. It provided enough of the Photoshop functionality (curves, etc.) that I often wouldn't need to launch Elements. Just as important, the reviews proclaimed Lightroom to be more responsive and less of a resource hog. _So what does all this have to do with **Lua?**_

Adobe Lightroom is programmed with C/C++/Objective-C and... **40% Lua**. An interpreted scripting language making up 40% of the code, and it feels more responsive and uses less resources?! That deserves a second look...

Adobe Lightroom puts Lua's coroutines to good use, keeping the user interface responsive while images visually rotate or exports process. Coroutines are a form of "cooperative multitasking," where the programmer controls task-switching rather then being preempted as with "real" threads.

I've taken a great interest in the Lua language. The more I learn, the more intrigued I am. This blog exists in order to share these findings with you. **Lua** is Portuguese for moon, and "lua nova" means _**new moon**_.

Welcome to Lua, welcome to the moon...
