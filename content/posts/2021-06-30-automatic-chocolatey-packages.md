---
title: "Automatic zero-maintenance Chocolatey package builds"
url: automatic-zero-maintenance-chocolatey-package-builds
aliases:
  - automatic-zero-maintenance-chocolatey-packages-builds
date: 2021-06-30T20:14:00+02:00
---

One of the important things of contributing to Open Source software is trying out bleeding edge builds.
For instance, while working on the new major release of [Apache Maven](https://maven.apache.org), I want to make use of it in my regular day job.
A process which easily installs the latest master build automatically would be convenient to have. 
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

Don't be fooled though, not only developer tools can be installed with it, but also applications like [Google Chrome](https://community.chocolatey.org/packages/GoogleChrome), [Zoom](https://community.chocolatey.org/packages/zoom) and [GIMP](https://community.chocolatey.org/packages/gimp)!

# Development
Before we can automatically update and build the package, we should develop an initial version first. 
This article will not dive deep into developing it, but I felt the article would not be complete without at least covering it a bit.
Another reason for not diving deep into the development part is because Chocolatey has [extensive documentation](https://docs.chocolatey.org/en-us/create/create-packages) itself.
I suggest you to follow these docs when you are serious about creating a package, it helped me out a great bit. 

One thing to realize is that, since we are building something for Windows, Powershell experience will come in handy. 
Only in the event where you package the binaries inside of the Chocolatey package, you might not need any of the following scripts. 
Otherwise, a Chocolatey package typically contains three scripts:
 - `chocolateyInstall.ps1` - This powershell script will be run when someone invokes `choco install <your-package>`. It will also run again when someone upgrades the package.
 - `chocolateyBeforeModify.ps1` - This script will run when someone upgrades or uninstalls the package. This allows developers to prepare an installation for modification. 
 - `chocolateyUninstall.ps1` - When a user uninstalls a package, this will be run after `chocolateyBeforeModify.ps1` (if it is present at all).

The installation script can download binaries from the internet and install it on a computer. 
Chocolatey offers modules which can help with this, for example `Install-ChocolateyZipPackage`. 
This function expects you to add a checksum of the file that will be downloaded during the install script.
With this, the user is protected against man-in-the-middle attacks or just faulty downloads. 

Other things will typically need to be done in these scripts, such as changing the environment variables and other things which completes the installation.
Take care, since Powershell scripts can pretty much change anything in your system, errors can be disastrous. 
To be safe, you can develop and test these scripts in a virtual machine. 

# Automation
Now that we have our first version of the Chocolatey package ready, we can start looking at how to automatically update it.
Chocolatey suggests to take a look at the [Chocolatey Automatic Package Updater Module](https://github.com/majkinetor/au/blob/master/README.md).
While it does look very useful, I didn't know of it before I built the maven-snapshot package. 
I definitely would advise you to read these docs, before building something on your own as I did. 
Might save you some time if it fits your requirements.

## Build system
To automatically run a script, you need a build system which can take care of this. 
As I've gotten some good experience building [Maven's workflow in GitHub Actions](https://github.com/apache/maven/actions), I have used that for this project as well.
Of course, any build system that has the ability to run jobs in a scheduled fashion could be used.

## What should be automated?
That differs per package, but in my case, I wanted to have a package which automatically contained the last successful Maven build from the master branch. 
The [Jenkins server](https://ci-builds.apache.org/job/Maven/job/maven-box/job/maven/job/master/) of the Apache Software Foundation builds the master branch of Maven everytime it changes.
This results in new distributions of snapshot versions that can freely be downloaded. 
How convenient! 
Jenkins also offers an API to fetch the history of the last run builds. 
Maven-snapshot's workflow basically does the following things:
1. Fetch the latest successful Maven build number and only continue when it's different from the last workflow run.
2. Download the binaries (in this case a ZIP archive) from Jenkins to calculate the sha256 checksum.
3. Update the download URL and checksum in the Chocolatey install script.
4. Update the version of the Chocolatey package in the `.nuspec` file to some version which is later than the current version (See the [versioning recommendations](https://docs.chocolatey.org/en-us/create/create-packages#versioning-recommendations)).
5. Build the Chocolatey package using `choco pack`.
6. Install the package and run `mvn --version` to verify that the correct version has been installed.
7. Push the newly built package to Chocolatey. To push your package you need an API key. Make sure to keep that hidden, otherwise other people will be able to push unwanted updates to your package... GitHub Actions offers [Encrypted secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) for this purpose.
8. Persist the changes to the Git repo. Other than the package scripts, it also updates the file keeping track of the latest installed package version used in step 1.

View the [source](https://github.com/MartinKanters/ChocolateyPackages/blob/master/.github/workflows/maven-snapshot-update.yml) here, and see it in _action_ [here](https://github.com/MartinKanters/ChocolateyPackages/actions).

## Chocolatey's testing and verification stages
After pushing the newly built package, it might take some time before it is publicly listed. 
In the meantime people can already use it, but they will have to specify the version with a flag in the install command: `choco install maven-snapshot --version=20210625.1907`.
It takes some time to get listed, because Chocolatey has some automated checks which verifies whether the package is correct.
One of the things the automated checks do is install and uninstall the package to make sure it works correctly.
Usually this is done in about half an hour.

Unfortunately, when your package is really new, it will also be checked manually by a Chocolatey moderator. 
This process really takes substantial amount of time, probably because there are too many packages and too little moderators.
For my package this often took more than a week and up to three weeks before it was approved. 
For packages that do not get updated often, this might be just fine.
The aim for maven-snapshot is to allow people to easily download the most recent snapshot build, so waiting for weeks is not desirable.
Luckily, after about one month and a half, the package became a [trusted package](https://docs.chocolatey.org/en-us/faqs#what-is-a-trusted-package), which means it skips manual tests. 

When a package gets uploaded and does not pass the stages, Chocolatey asks you to create a correct package and reupload that. 
For an automated process, that is quite annoying... especially when a later snapshot build has already been packaged and uploaded in the meantime on a newer version number. 
In that case, depending on where it failed in the automatic testing process, you can either unlist the package yourself or leave a note to a moderator asking for it to be removed.

After setting up everything, you have managed to create an automatic pipeline which builds, tests and uploads a Chocolatey package, without any maintenance whatsoever!
Congrats!

# Wrapping up
So, in this article I explained how you can automate your build pipeline around Chocolatey packages.
I used the Chocolatey package for [maven-snapshot](https://chocolatey.org/packages/maven-snapshot) as example. 
Although it's developed by me, I took a lot of inspiration from [@mthmulders](https://twitter.com/mthmulders)'s [homebrew package](https://github.com/mthmulders/homebrew-maven-snapshot) and the original [Maven Chocolatey package](https://community.chocolatey.org/packages/maven).
The article mentions how to use a pipeline tool like GitHub Actions to detect new versions, tests, packs and uploads them to Chocolatey. 
Then Chocolatey continues the testing progress, before listing them publicly. 

Even though I was not experienced in Powershell or Chocolatey before starting this project, I still managed to create this in under a week. 
I bet you can do better!

_... Oh and one more Chocolatey tip, make sure to create some email filters for their mail notifications. They can be pretty noisy. :)_

I hope this helped, or at least was a nice read. Either way, I would like to thank you for reading!
