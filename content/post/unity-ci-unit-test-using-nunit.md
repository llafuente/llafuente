+++
draft = false
title = "Unity CI: unit test using NUnit (Part I/Many)"
date = "2016-11-18T21:56:49+01:00"
tags = [ "c#", "ci", "unity", "testing" ]
categories = [ "Unity" ]
series = [ "Unity CI" ]
summary = """
First part of an unkown journy to setup a full CI on windows, linux and OSX.

In this episode we will focus on travis-ci setup with OSX"""

+++

CI Setup for Unity projects is far more difficult than I would imagine.

Mostly because the documentation is mostly for old version of Unity, it's like
everybody already knows that NUnit is bundled inside Unity and there is no
problems anymore... But when you are building a package you should check
many OSs and if possible many Unity versions.

These are my objectives for
[unity-platformer](https://github.com/llafuente/unity-platformer).

* OS matrix: Windows, Linux, OSX
* Unit test on any of those OSs
* Integration test on all
* Unity version: 5.5
* Do not include external libraries in the repo (like UnityTestTools)

In this epidode we focus on OSX unit test using
[travis-ci](https://travis-ci.org).

Why not Linux ?

* OSX Editor is +500mb. Linux is about 2gb
* Linux build can't be trusted at this moment. I have many problems with the
editor I expect problems with CI, will see.

For integration test we will use [Appveyor](https://www.appveyor.com).


### Project configuration

Project configuration is really simple at this point. Just three files

#### <i class="fa fa-file-o" aria-hidden="true"></i> ./.travis.yml

{{< highlight yaml >}}
language: objective-c
osx_image: xcode7.3
#rvm:
#- 2.1.2
before_install:
- chmod a+x ./Scripts/ci-osx-install.sh
- chmod a+x ./Scripts/ci-osx-test.sh
install:
- ./Scripts/ci-osx-install.sh
script:
- ./Scripts/ci-osx-test.sh
{{< /highlight >}}

#### <i class="fa fa-file-o" aria-hidden="true"></i> ./Scripts/ci-osx-install.sh

{{< highlight bash >}}
#! /bin/sh

BASE_URL=http://beta.unity3d.com/download
HASH=03364468e96e
VERSION=5.5.0b11
UNITY_TEST_TOOL_BRANCH=5.5

download() {
  file=$1
  url="$BASE_URL/$HASH/$package"

  echo "Downloading from $url: "
  curl -o `basename "$package"` "$url"
}

install() {
  package=$1
  download "$package"

  echo "Installing "`basename "$package"`
  sudo installer -dumplog -package `basename "$package"` -target /
}

install "MacEditorInstaller/Unity-$VERSION.pkg"

curl -o "UnityTestTools.tar.bz2" "https://bitbucket.org/Unity-Technologies/unitytesttools/get/${UNITY_TEST_TOOL_BRANCH}.tar.bz2"
UNITY_TEST_TOOLS_PATH=$(tar xvjf "UnityTestTools.tar.bz2" 2>&1 | tail -n 1 | cut -c3- | cut -d'/' -f1)
mv "${UNITY_TEST_TOOLS_PATH}/Assets/UnityTestTools" "./Assets/UnityTestTools"

{{< /highlight >}}

#### <i class="fa fa-file-o" aria-hidden="true"></i> ./Scripts/ci-osx-test.sh

{{< highlight bash >}}
#!/bin/sh

/Applications/Unity/Unity.app/Contents/MacOS/Unity \
  -batchmode \
  -nographics \
  -projectPath $(pwd) \
  -runEditorTests \
  -editorTestsResultFile $(pwd)/unit-test-results.xml \
  -editorTestsFilter YOUR_NAMESPACE \
  -logFile $(pwd)/log-unit-test.txt

cat $(pwd)/log-unit-test.txt
cat $(pwd)/unit-test-results.xml
{{< /highlight >}}

*NOTICE*: Why select your namespace? (`-editorTestsFilter YOUR_NAMESPACE`)

Remember that we are installing UnityTestTools, that has some unit test
examples. We don't want to execute them, mostly because they fail.

### Test other Unity versions

How to search HASH & VERSION ?

Stable versions

* https://unity3d.com/get-unity/download/archive
* `BASE_URL=http://netstorage.unity3d.com/unity`

Beta versions

* https://unity3d.com/unity/beta/archive
* `BASE_URL=http://beta.unity3d.com/download`

Check those dropdowns and copy those links, contains the HASH/VERSION needed.

Here are some :)

#### 5.3

{{< highlight bash >}}
BASE_URL=http://netstorage.unity3d.com/unity
HASH=c347874230fb
VERSION=5.3.7f1
UNITY_TEST_TOOL_BRANCH=5.3
{{< /highlight >}}

####  5.4

{{< highlight bash >}}
BASE_URL=http://netstorage.unity3d.com/unity
HASH=01f4c123905a
VERSION=5.4.3f1
# There is no branch 5.4 in UnityTestTools, 5.3 should be compatible.
UNITY_TEST_TOOL_BRANCH=5.3
{{< /highlight >}}

#### beta 5.5

{{< highlight bash >}}
BASE_URL=http://beta.unity3d.com/download
HASH=03364468e96e
VERSION=5.5.0b11
UNITY_TEST_TOOL_BRANCH=5.4
{{< /highlight >}}

### Coding problems.

Your code style need to be "adapted" to be testable.

#### Do not use *internal* (C# keyword)

I used to avoid `[HideInInspector] public` but unit test
lives in the Editor DLL, and that means it's not in the same DLLs that your
game code, so unit test won't have access to those properties.

#### Replace Debug.LogWarning/LogError with Assertions

Debug.* do not throw an error that NUnint can catch. It was fine while manually
testing to see those errors in the Console but Assertions has the same effect
and it's unit test friendly.

[UnityEngine.Assertions](https://docs.unity3d.com/ScriptReference/Assertions.Assert.html)

#### RequireComponent can't be trusted

RequireComponent only works on Editor/Unity runtime.

Most of the times you will use `GetComponent<Type>()` result thinking
it's safe... but it's not! I mean it!
`AddComponent<Type>()` do not add required components while unit testing.
And that lead `GetComponent<Type>()` to *NullReferenceException*.

Always Assert after `GetComponent<Type>()`

{{< highlight csharp >}}
body = GetComponent<BoxCollider2D>();
Assert.IsNotNull(body, "BoxCollider2D is required: " + gameObject.name);
{{< /highlight >}}

#### Unity events should be public

Like before, Unity don't require `Start`, `OnEnable` (etc.) to be *public*,
but when you are unit testing you are outside the "Unity runtime".
Nobody will call `Start` for you.

That's the main reason that people think MonoBehaviours cannot be unit tested...
nonsene! It just means that you have to manually emulate Unity.

Another good practice, is to check object integrity (with Assertions) at `Start`
method. I found out that Start in better than `OnEnable`, but it's mostly a
personal preference.

### Tips / Snippets

#### Initialize LayerMask

{{< highlight csharp >}}
LayerMask lm = (1 << 2) | (1 << 3); // will check 2nd (2) or 3rd (4) layer
{{< /highlight >}}

#### Do not use Assert.Pass();

`Assert.Pass();` Creates even more XML tags in the already terminal-unreadable
result file.

#### Use external Loggin system

Debug.Log/Warning/Error will generate at least 20 lines in unity logFile
because it include the backtrace, totally unreadable.
