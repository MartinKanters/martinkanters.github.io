---
title: "Automatically managing Chocolatey packages"
url: automatically-managing-chocolatey-packages
date: 2021-06-01T00:00:00+02:00 # TODO
draft: true
---

One of the important things of contributing to Open Source software is trying out bleeding edge builds.
For instance, while working on the new major release of [Apache Maven](https://maven.apache.org), I want to make use of it in my daily work life.
A process to easily install the latest master build would be convenient to have. 
My friend and colleague [@mthmulders](https://twitter.com/mthmulders) made a [homebrew package](https://github.com/mthmulders/homebrew-maven-snapshot) for it. 
Unfortunately, as I'm currently on Windows, that does not help me.
That's why I decided to build a [Chocolatey package](https://chocolatey.org/packages/maven-snapshot) for it. 
This article zooms in on how this package is managed without me having to touch it.

# What is Chocolatey

[Chocolatey](http://chocolatey.org/) is a tool for Windows for installing, upgrading and uninstalling applications from the command line.
It can roughly be compared to apt (popular on Debian Linux distributions) or Homebrew (popular on MacOS).
Perhaps some examples will clear up some confusion. 
The following block installs the [Maven-snapshot](https://community.chocolatey.org/packages/maven-snapshot) Chocolatey package:

```
> choco install maven-snapshot -Y
Chocolatey v0.10.15
Installing the following packages:
maven-snapshot
By installing you accept licenses for the packages.
Progress: Downloading maven-snapshot 20210612.1109... 100%

maven-snapshot v20210612.1109 [Approved]
maven-snapshot package files install completed. Performing other installation steps.
Downloading Maven Snapshot
  from 'https://ci-builds.apache.org/job/Maven/job/maven-box/job/maven/job/master/169/artifact/org/apache/maven/apache-maven/4.0.0-alpha-1-SNAPSHOT/apache-maven-4.0.0-alpha-1-SNAPSHOT-bin.zip'
Progress: 100% - Completed download of ...\chocolatey\maven-snapshot\20210612.1109\apache-maven-4.0.0-alpha-1-SNAPSHOT-bin.zip (9.14 MB).
Download of apache-maven-4.0.0-alpha-1-SNAPSHOT-bin.zip (9.14 MB) completed.
Hashes match.
Extracting ...\chocolatey\maven-snapshot\20210612.1109\apache-maven-4.0.0-alpha-1-SNAPSHOT-bin.zip to C:\ProgramData\chocolatey\lib\maven-snapshot...
C:\ProgramData\chocolatey\lib\maven-snapshot
C:\Users\Martin\.m2
Environment Vars (like PATH) have changed. Close/reopen your shell to
 see the changes (or in powershell/cmd.exe just type `refreshenv`).
 The install of maven-snapshot was successful.
  Software installed to 'C:\ProgramData\chocolatey\lib\maven-snapshot'

Chocolatey installed 1/1 packages.
 See the log for details (C:\ProgramData\chocolatey\logs\chocolatey.log).
```

Afterwards it can be upgraded and uninstalled with the following commands:

```
choco upgrade maven-snapshot -Y
choco uninstall maven-snapshot -Y
```

Do not be fooled though, not only developer tools can be installed with it, but also applications like [Google Chrome](https://community.chocolatey.org/packages/GoogleChrome), [Zoom](https://community.chocolatey.org/packages/zoom), [GIMP](https://community.chocolatey.org/packages/gimp) and many more!






--- TODO: Handling the client secret  
--- TODO: Shoutout to official choco package

--- TODO: Automatic testing/verification  
--- TODO: What to do when flakiness occurs  