+++
draft = false
title = "Ionic 3 Development Environment"
date = "2019-01-22T11:52:00+01:00"
tags = ["Virtual machines", "Ionic"]
categories = ["Ionic", "FrontEnd"]
series = []
images = []
description = ""
summary = """
How to setup your Ionic 3 Development Environment with Android on Windows.
"""


+++

### Disclaimer

The main purpose for this article was to run Ionic 3 inside a VM.
Everything was working as expected until I tried to run the Android Emulator
and face reality:

`Android Emulator does not support nested virtualization`

I did built some projects, and also test it in real devices, but cannot emulate
them...

Microsoft Virtual Machine expires in three months,
so I cannot upload my VM and use it in the future... :sob:
Also notice that all scripts will work directly on your machine if you
need emulation.

### VM (optional)

First need to [download the official windows VM](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines), it will take some time.

You can choose any virtualizacion:

* VMWare
* Hyper-V
* VirtualBox
* Parallels


Before install I do another optional step.
Configure `Shared folder` and copy the following scripts,
then just execute them as administrator.


### Install dependencies

We will handle everything with [chocolatey](https://chocolatey.org/)

Start an Administrator CMD

![Start an Administrator CMD in windows 10](windows-10-run-cmd-as-administrator.png)

{{< highlight cmd >}}
@rem install chocolatey
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

@rem install android-related
choco install -y androidstudio --params "/AddToDesktop"
choco install -y android-sdk
choco install -y jdk8

@rem install ionic-related
choco install -y nodejs-lts
@rem needed for node-sass -> node-gyp
choco install -y python2
{{< /highlight >}}

Close and start an administrator CMD again to load new environment vars.


#### Android

{{< highlight cmd >}}
@rem sdkmanager --list
@rem this may take a few hours...
@rem common staff
sdkmanager "tools" "emulator" "platform-tools"
sdkmanager "extras;google;m2repository" "extras;android;m2repository"

@rem sdk version, images and build.
sdkmanager "platforms;android-28" "build-tools;28.0.3"
sdkmanager "system-images;android-28;default;x86_64"

@rem accept all licenses
sdkmanager --licences
{{< /highlight >}}

#### Ionic

{{< highlight cmd >}}
npm config set python python
npm install -g ionic cordova
{{< /highlight >}}

#### sublime 3 (optional)

This is my editor of choice. Install using powershell:

{{< highlight powershell >}}
choco install -y sublimetext3 /AddToDesktop
cd "$env:userprofile\AppData\Roaming\Sublime Text 3\Installed Packages"
wget -o "Package Control.sublime-package" "https://packagecontrol.io/Package%20Control.sublime-package"
{{< /highlight >}}

### Install under proxy ?

#### chocolately

before first usage:

{{< highlight powershell >}}
$env:chocolateyProxyLocation = 'http://<proxy-host>:<proxy-port>'
$env:chocolateyProxyUser = '<proxy-user>'
$env:chocolateyProxyPassword = '<proxy-password>'


choco config set proxy <proxy-host>:<proxy-port>
choco config set proxyUser <proxy-user>
choco config set proxyPassword <proxy-password>
{{< /highlight >}}

**Cleanup**

{{< highlight cmd >}}
choco config unset proxy
choco config unset proxyUser
choco config unset proxyPassword
{{< /highlight >}}

#### sdkmanager

Add to every sdkmanager command:

{{< highlight cmd >}}
--no_https --proxy=http --proxy_host=<proxy-host> --proxy_port=<proxy-port>
{{< /highlight >}}

Do you need username and password? [read: How to set proxy for android sdk manager?
](https://stackoverflow.com/questions/42296708/how-to-set-proxy-for-android-sdk-manager/51973575#51973575)


#### npm

before first usage:

{{< highlight cmd >}}
npm config set proxy "http://<proxy-user>:<proxy-password>@<proxy-host>:<proxy-port>"
npm config set https-proxy "http://<proxy-user>:<proxy-password>@<proxy-host>:<proxy-port>"
{{< /highlight >}}


### Test our first Ionic App

#### Manage android virtual devices

**create**

* `--name` : name of the android virtual device.

* `--package` : package to use.

    `sdkmanager <system-images;*****>` to install the packages.

    `avdmanager list target`, list pacakages.

* `-d` : device id for hardware profile.

    `avdmanager list` for the list of devices

{{< highlight cmd >}}
avdmanager create avd --name Emulator-Api28-Google --package "system-images;android-28;default;x86_64" -d 2
{{< /highlight >}}


**list**

{{< highlight cmd >}}
avdmanager list avd
{{< /highlight >}}

**delete**

{{< highlight cmd >}}
avdmanager delete avd -n Emulator-Api28-Google
{{< /highlight >}}


#### Ionic App

{{< highlight cmd >}}
ionic start testApp tabs
cd testApp

@rem test on a desktop browser
ionic serve

@rem build for android
set DEBUG=1
ionic cordova build android --prod --verbose

@rem run on a real/emulated device
ionic cordova run android --livereload
{{< /highlight >}}



### references

* https://gist.github.com/search?utf8=%E2%9C%93&q=avdmanager+create+avd

* https://gist.github.com/search?l=Shell&q=sdkmanager

* [Ionic CLI](https://ionicframework.com/docs/cli/)

* [Ionic cordova build](https://ionicframework.com/docs/cli/cordova/build/)

* [Ionic cordova run](https://ionicframework.com/docs/cli/cordova/run/)
