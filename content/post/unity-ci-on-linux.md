+++
draft = false
title = "Unity CI: Linux (Part II/Many)"
date = "2017-01-16T22:56:49+01:00"
tags = [ "c#", "ci", "unity", "testing" ]
categories = [ "Unity" ]
series = [ "Unity CI" ]
images = []
description = ""
summary = """
Second part of an unknown journey to set up a full CI on Windows, Linux and OSX.

In this episode we will focus on travis-ci setup with Linux while keeping OSX.
"""

+++

As always the project is
[unity-platformer](https://github.com/llafuente/unity-platformer).
Right now has OSX, Linux working, windows is yet wip.


First we will change out `.travis.yml` to run a Linux/OSX matrix

#### <i class="fa fa-file-o" aria-hidden="true"></i> ./.travis.yml

{{< highlight yaml >}}
language: csharp

matrix:
  include:
    - os: linux
      dist: trusty
      sudo: required
    - os: osx
      osx_image: xcode7.3

before_install:
- chmod a+x ./Scripts/ci-osx-install.sh
- chmod a+x ./Scripts/ci-osx-test.sh
- chmod a+x ./Scripts/ci-linux-install.sh
- chmod a+x ./Scripts/ci-linux-test.sh

install:
- if [ "$TRAVIS_OS_NAME" == "osx" ]; then ./Scripts/ci-osx-install.sh; fi;
- if [ "$TRAVIS_OS_NAME" == "linux" ]; then ./Scripts/ci-linux-install.sh; fi;

script:
- if [ "$TRAVIS_OS_NAME" == "osx" ]; then ./Scripts/ci-osx-test.sh; fi;
- if [ "$TRAVIS_OS_NAME" == "linux" ]; then ./Scripts/ci-linux-test.sh; fi;
{{< /highlight >}}


And now the install/test scripts are mostly the same that OSX.

#### <i class="fa fa-file-o" aria-hidden="true"></i> ci-linux-install.sh

{{< highlight bash >}}
#!/bin/sh

set -ex

UNITY_TEST_TOOL_BRANCH=5.5

wget -O unity-editor.deb 'http://beta.unity3d.com/download/59c25c92588f/unity-editor_amd64-5.5.0xp1Linux.deb'
# install and fix dependencies broken by installing Unity
# in the same command, because dpkg will fail...
sudo dpkg -i unity-editor.deb || sudo apt-get -y -f install


curl -o "UnityTestTools.tar.bz2" "https://bitbucket.org/Unity-Technologies/unitytesttools/get/${UNITY_TEST_TOOL_BRANCH}.tar.bz2"
UNITY_TEST_TOOLS_PATH=$(tar xvjf "UnityTestTools.tar.bz2" 2>&1 | tail -n 1 | cut -c1- | cut -d'/' -f1)
mv "${UNITY_TEST_TOOLS_PATH}/Assets/UnityTestTools" "./Assets/UnityTestTools"
{{< /highlight >}}

#### <i class="fa fa-file-o" aria-hidden="true"></i> ci-linux-test.sh

{{< highlight bash >}}
#!/bin/sh

set -ex

/opt/Unity/Editor/Unity \
  -batchmode \
  -nographics \
  -projectPath $(pwd) \
  -runEditorTests \
  -editorTestsResultFile $(pwd)/unit-test-results.xml \
  -editorTestsFilter UnityPlatformer \
  -logFile $(pwd)/log-unit-test.txt

cat $(pwd)/log-unit-test.txt
cat $(pwd)/unit-test-results.xml
{{< /highlight >}}


### TODO

I know now when I broke something (it's broken right now) but the CI report
don't tell me where, so in the future I will add something that parse
`ǸUnit XML` and print something relevant and realable into console.
