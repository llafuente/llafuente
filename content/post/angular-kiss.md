+++
draft = false
title = "Angular. Keep it simple."
date = "2017-02-15T19:56:35+01:00"
tags = ["Angular2"]
categories = ["Angular2", "FrontEnd"]
series = []
images = []
description = ""
summary = """
When business logic is complicated simplify the rest, do not try to
complicate things more than it should...
"""

+++

When business logic is complicated simplify the rest, do not try to
complicate things more than it should...

This must be your ninja way in Angular. Don't do anything fancy, this may not be
obvius but TypeScript is NOT Java, don't follow the same naming structure.
Packages are not needed, you only damage your JS/TS developers.

At follow I will summarize some things i found myself as Good-Practice that are
against Good-practices.

# Split your applications in small modules.

This is a very good approach that became a snowball of death and destruction.
I'm not saying you shouldn't split your application, you should but only if you
have `@angular/core` imports. If you need something from your application, then
it's part of your main module.


Until you have login / user session.
Then you have a Module with a Service that it's shared by all modules.

When you split your module you have to import many common things, like
`BrowserModule`. In fact module declaration because 80% non-important,
non-reusable, pure verbose. And you start to think, I don't mind... it's just
a declaration... YOU SHOULD! because that means all test are verbose! I see many
projects that the first 100 lines of every spec file are useless because usually
just change a single line.
