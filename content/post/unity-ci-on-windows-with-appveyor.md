+++
draft = true
title = "Unity CI on Windows with Appveyor"
date = "2016-12-01T12:20:11+01:00"
tags = [ "c#", "ci", "unity", "testing" ]
categories = [ "Unity" ]
series = [ "Unity CI" ]
images = []
description = ""
summary = """
Second part of an unknown journey to set up a full CI on Windows, Linux and OSX.

In this episode we will focus on Appveyor setup with windows.
"""

+++


Oh! Windows! My beloved Windows!, no jokes! After a long time working on
Windows and Linux environments I started to miss Windows more and more.
In the end, I decide to be more productive and work on Windows and run
Linux virtual machine inside. So I have the best of both worlds!

But sometimes, unexpected things happen, today is one of those. People do
something for windows just... for no reason.

{{< quote >}}
Unity command line in windows require a PRO license or a manually
inserted personal license.
{{< / quote >}}


Here are some thing I try before I realise the truth: It wasn't my fault.

* Use `-force-free`
  From documentation `Make the Editor run as if there is a free Unity license on the machine, even if a Unity Pro license is installed.`

* Use `-username <username> -password <password> -serial <serial>`

* Run Unity.exe before, kill it and run it in batchmode


After that I started to look in the Unity forums. And found is not possible.
And start thinking about **the work-around route**! (it's like Naruto's ninja
way but for programmers)

My idea was to emulate user input to introduce the license, then test like
in any other platform. Tools: [Autohotkey](https://autohotkey.com/)
([Latest](https://www.autohotkey.com/download/ahk-install.exe)) and
[MacroCreator](http://www.macrocreator.com/)
([v5.0.5](https://github.com/Pulover/PuloversMacroCreator/releases/download/v5.0.5/MacroCreator-setup.exe)).

I'm not going to show how to create the script... just use MacroCreator.
But there is one thing you should notice before doing it... Autohotkey depens
on screen resolution to position the mouse.

Appveyor support resize the screen so no problem!

{{< highlight yaml >}}
- ps: iex ((new-object net.webclient).DownloadString('https://raw.githubusercontent.com/appveyor/ci/master/scripts/set-screenresolution.ps1'))
- ps: Set-ScreenResolution 1280 1024
{{< /highlight >}}


Also you need to store your user and password on Appveyor for that use
[encripted secure variables](https://www.appveyor.com/docs/build-configuration/#secure-variables)

These are mines (*below is the full appveyor.yml*)

{{< highlight yaml >}}
environment:
  UNITY_USERNAME:
    secure: ZUUZL2M/v3MaxAafHfCt3fgNhxfYpVh8B3PVtifztf0=
  UNITY_PASSWORD:
    secure: 3X2Sg2Rqe3A+yzguihwNtA==
{{< /highlight >}}


----

[Send](https://autohotkey.com/docs/commands/Send.htm)
[Click](https://autohotkey.com/docs/commands/Click.htm)
[MouseClick](https://autohotkey.com/docs/commands/MouseClick.htm)
[MouseMove](https://autohotkey.com/docs/commands/MouseMove.htm)


${env:UNITY_USERNAME}
${env:UNITY_PASSWORD}
