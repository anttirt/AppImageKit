# AppImageKit [![Build Status](https://travis-ci.org/probonopd/AppImageKit.svg?branch=master)](https://travis-ci.org/probonopd/AppImageKit) [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/probonopd/AppImageKit?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

Copyright (c) 2004-15 Simon Peter <probono@puredarwin.org> and contributors.

The AppImage format is a standardized format for packaging applications in a way that allows them to
run on target systems without further modification. This document describes the AppImage format.

AppImageKit  contains  a  concrete  implementation  of  the  AppImage  format  and  provides  tools  for
conveniently handling AppImages.

This document is not a formal specification, since the AppImage format is not frozen yet but in the process
of being specified more formally. However, this document is intended to describe the philosophy behing
the AppImage format and the concrete implementation. Contributors are encouraged to comment on this document and propose formal format descriptions.

## Motivation

Historically, UNIX and Linux systems have made it easy to procure source code, however they have made it comparably difficult to use ready-made software in binary form. Especially the Free Software and GNU movements stress the fact that everyone should be free to get the source code. This mode of operation has worked well as long as the user base of these operating systems was largely comprised of technically advanced users. With the widespread adoption of easy-to-use desktop operating systems such as Ubuntu, the user base became less technically-minded and more application-centric.

Package mangers were introduced to mitigate the complexities of dealing with source code by providing libraries of precompiled packages from repositories maintained by distributors or third parties. However, the introduction of package mangers did not drastically reduce compexities or provide robustness - they merely stacked a management layer on top of an already complex system, effectively preventing the user from manipulating installed software directly.

Other systems, most prominently Windows and the legacy Macintosh operating system, have made it relatively simple for independent software publishers (ISPs) to distribute software and for end-users to procure and install software from said ISPs, without any instances (such as distributors or AppStores) between the two parties.

With the introduction of Mac OS X, arguably the first UNIX-based operating system with widespread mass adoption to a non-technical user base, Apple blended traditional UNIX aspects (such as maintaining a traditional filesystem hierarchy, including /bin, /usr, /lib directories) with common "desktop" approaches (such as "installing" an application by dragging it to the hard disk). While Apple uses a package manager-like approach for managing the base operating system and its updates, it does not do so for the user applications.

Open Source operating systems, such as the most prominent Linux distributions, mostly use package mangers for everything. While this is perceived superior to Windows and the Mac by many Linux enthusiasts, it also creates a number of disadvantages:

 1. __Centralization__ Some organization decides what is "in" a distribution and what is not. By definition, software "in" a distribution is easier to install and manage that software that is not.
 2. __Duplication of effort__ In traditional systems, each application is compiled specifically for each target operating system. This means that one piece of software has to be compiled many, many times on many, many systems using much, much power and time
 3. __Need to be online__ Most package managers are created with connected computers in the mind, making it really cumbersome to "just fetch an app" on an online system, and copy it over to another system that is not connected to the Internet.
 
A critical distinction between the approach known from Windows and the Mac and the one known from UNIX and Linux is the "platform": While Windows and the Mac are seen as platforms to run software on, most Linux distributions see themselves as the system that includes the applications.

While this leads to a number of advantages that have been frequently reiterated, it also poses a significant number of challenges:

 1. __No recent apps on mature operating systems__. In most distributions, you get only the version that was recent at the time when the distribution was created. For example, if you use Ubuntu Gutsy then you are stuck forever with the software that was recent at the time when Ubuntu Gutsy was compiled. Even if Firefox might have progressed by several versions in the meantime, you cannot get more recent apps than what was available back when the distribution was put together. That is like if you'd get only software from 2001 when you use Windows XP. In the traditional model, the user has to decide: Either use a mature base operating system but be locked out of recent apps (e.g., using Ubuntu LTE), or be forced to update the base operating system to the latest bleeding edge version in order to get the recent apps (e.g., using Debian Sid). This situation is clearly not optimal, since the common desktop user would prefer to hardly touch the base operating system (maybe update it every other year or so) but always get the latest apps.
 2. __No simple way to use multiple versions in parallel__. Most package managers do not allow you to have more than one version of an app installed in parallel. Hence you have no way to simply try out the latest version of an app without running the risk that it might not be easy to switch back to the older version, especially if the older version is no longer available in your distribution (e.g, old versions get removed from Debian Sid as soon as a newer version appears). This is especially annoying if you would simply like to try out a few things before you decide whether to use the old or the new version.
 3. __Not easy to move an app from one machine to another__. If you've used an app on one machine and decide that you would like to use the same app either under a different base operating system (say, you want to use OpenOffice on Fedora after having used it on Ubuntu) or if you would simply take the app from one machine to another (say from the desktop computer to the netbook), you have to download and install the app again (if you did not keep around the installation files and if the two operating systems don't share the exact same package format - both of which is rather unlikely).

The AppImage format has been created with specific objectives in mind.

 1. __Be Simple__. AppImage is intended to be a very simple format that is easy to understand, create, and manage.
 2. __Maintain binary compatibility__. AppImage is a format for binary software distribution. Software packaged as AppImage is intended to be as binary-compatible as possible with as many systems as possible. The need for (re-)compilation of software should be greatly reduced.
 3. __Be distribution-agostic__. An AppImage should run on all base operating systems (distributions) that it was created for (and later versions). For example, you could target Ubuntu 9.10, openSUSE 11.2, and Fedora 13 (and later versions) at the same time, without having to create and maintain separate packages for each target system.
 4. __Remove the need for installation__. AppImages contain the app in a format that allows it to run directly from the archive, without having to be installed first. This is comparable to a Live CD. Before Live CDs, operating systems had to be installed first before they could be used.
 5. __Keep apps compressed all the time__. Since the application remains packaged all the time, it is never uncompressed on the hard disk. The computer uncompresses the application on-the-fly while accessing it. Since decompression is faster than reading from hard disk on most systems, this has a speed advantage in addition to saving space. Also, the time needed for installation is entirely removed.
 6. __Allow to put apps anywhere__. AppImages are "relocateable", this allowing the user to store and exectute them from any location (including CD-ROMs, DVDs, removable disks, USB sticks).
 7. __Make applications read-only__. Since AppImages are read-only by design, the user can be reasonably sure that an app does not modify itself during operation.
 8. __Do not require recompilation__. AppImages must be possible to create from already-existing binaries, without the need for recompilation. This greatly speeds up the AppImage creation process, since no compiler has to be involved. This also allows third parties to package closed-source applications as AppImages. (Nevertheless, it can be beneficial for upstream application developers to build from source speficially for the purpose of generating an AppImage.)
 9. __Keep base operating system untouched__. Since AppImages are intended to run on plain systems that have not been specially prepared by an administrator, AppImages may not require any unusual preparation of the base operating system. Hence, they cannot rely on special kernel patches, kernel modules, or any applications that do not come with the targeted distributions by default.
 10. __Do not require root__. Since AppImages are intended to be run by end users, they should not reqiure an administrative account (root) to be installed or used. They may, however, be installed by an administrator (e.g., in multi-user scenarios) if so desired.

The key idea of the AppImage format is __one app = one file__. Every AppImage contains an app and all the files the app needs to run. In other words, each AppImage has no dependencies other what is included in the targeted base operating system(s). While it would theoretically be possible to create rpm or deb packages in the same way, it is hardly ever done. In contrast, doing so is strongly encouraged when dealing with AppImages and is the default use case of the AppImage format.

In short: An AppImage is for an app what a Live CD is for an operating system.

## AppImage Format overview

An AppImage is an ISO 9660 file with zisofs compression containing a minimal AppDir (a directory that contains the app and all the files that it requires to run which are not part of the targeted base operating systems) and a tiny runtime executable embedded into its header. Hence, an AppImage is both an ISO 9660 file (that you can mount and examine) and an ELF executable (that you can execute).

When you execute an AppImage, the tiny embedded runtime mounts the ISO file, and executes the app contained therein.

A minimal AppImage could potentially look like this:

```
----------------------------------------------------------
|            |                                           |
| ELF        | ISO9660 zisofs compressed data containing |
| embedded   | AppRun                                    |
| in ISO9660 | .DirIcon                                  |
| header     | SomeAppFile                               |
|            |                                           |
---------------------------------------------------------
```

 * AppRun is the binary hat is executed when the AppImage is run
 * .DirIcon contains a 48x48 pixel PNG icon that is used for the AppImage
 * SomeAppFile could be some random file that the app reqires to run

However, in order to allow for automated generation, processing, and richer metadata, the AppImage format follows a somewhat more elaborate convention:

```
----------------------------------------------------------
|            |                                           |
| ELF        | ISO9660 zisofs compressed data containing |
| embedded   | AppRun                                    |
| in ISO9660 | appname.desktop                           |
| header     | usr/bin/appname                           |
|            | usr/lib/libname.so.0                      |
|            | usr/share/icons/*/48x48/apps/iconname.png |
|            | usr/share/appname/somehelperfile          |
|            | .DirIcon                                  |
|            |                                           |
---------------------------------------------------------
```

 * The `runtime` ELF embedded in the ISO9660 header always executes the file called `AppRun` which is stored inside the ISO9660 file.
 * The file `AppRun` inside the ISO9660 file is not the actual executable, but instead a tiny helper binary that finds and exectues the actual app. Generic `AppRun` files have been implemented in bash and C as parts of AppImageKit. The C version, `AppRun.c`, is generally preferred as it is faster and more portable.
 * AppRun usually does not contain hardcoded information about the app, but instead retrieves it from the file `appname.desktop` that follows the Desktop File Specification.

A minimal appname.desktop file that would be sufficient for AppImage would need to contain

```
[Desktop Entry]
Name=AppName
Exec=appname
Icon=iconname
```
This desktop file would tell the `AppRun` executable to run the executable called `appname`, and would specify `AppName` as the name for the AppImage, and `iconname.png` as its icon.

However, it does not hurt if the desktop file contains additional information. Should it be desired to provide additional metadata with an AppImage, the desktop file could be extended with `X-AppImage-...` fields as per the Desktop File Specification. Usually, desktop files provided in deb or rpm archives are suitable to be used in AppImages. However, abolute paths in the `Exec=` statement are not supported by the AppImage format.

The AppImage contains the usual `usr/` hierarchy (following the File Hierarchy Standard). In the concrete example from the desktop file above, the AppRun executable would look for `usr/bin/appname` and would execute that. Also, AppImageKitAssistant (a tool used to create AppImages easily) would look for `usr/share/icons/*/48x48/iconname.png` and use that as the `.DirIcon` file, effectively making it the icon of the AppImage.

The app must be programmed in a way that allows for relocation. In other words, the app must not have hardcoded paths such as `/usr/bin`, `/usr/share`, `/etc` inside the binary. Instead, it must use relative paths, such as `./bin`. 

Since most binaries contained in deb and rpm archives generally are not created in a way that allows for relocation, they need to be either changed and recompiled (e.g., using the binreloc framework), or the binaries need to be patched. As recompiling is not convenient in most cases, AppRun changes to the `usr/` directory prior to executing the app, enabling the app to specify all paths relative to the AppImage's `usr/` directory. This allows one to use patched binaries (where the string `/usr` has been replaced with the same-length string `././`, which means "current directory"). Note that if you use the `././` patch, then your app must not use chdir, or otherwise it will break.

Note that the AppImage format has been conceived to facilitate the conversion of deb and rpm packages into the AppImage format with minimal manual effort. Hence, it contains some conventions in addition to those specified by the AppDir format, to which it is compatible to the extent that an unpacked AppImage can be used as an AppDir with the ROX Filer.

## AppImageKit

The AppImage format is complemented by a suite of tools called AppImageKit that provide a concrete sample implementation of the ideas expressed in the format, and that can greatly simplify dealing with AppImages.

Currently the AppImageKit contains (among others): 
 * __AppImageAssistant__, a CLI and GUI app that turns an AppDir into an AppImage.
 * __AppRun__, the executable that finds and executes the app contained in the AppImage.
 * __runtime__, the tiny ELF binary that is embedded into the header of each AppImage. AppImageKit automatically embeds the runtime into the AppImages it creates.
 * __apt-appdir__, a command line tool running on Ubuntu that turns packaged software into AppDirs. This tool can be used to semi-automatically prepare AppDirs that can be used as the input for AppImageAssistant. Note that while apt-appdir has been written for Ubuntu, it should also run on debian and could be ported to other distributions as well, then using the respective package managers insted of apt-get.

AppImageKit also contains additional tools and helpers.

## Building AppImageKit

Use an old system for building (at least 2-3 years old) to ensure the binaries run on older systems too. (Or use LibcWrapGenerator to ensure the build products run on older glibc versions; but to get started it might be the easiest to use a not too recent build system.)

```bash
sudo apt-get update ; sudo apt-get -y install libfuse-dev libglib2.0-dev cmake git libc6-dev binutils fuse # debian, Ubuntu
yum install git cmake binutils fuse glibc-devel glib2-devel fuse-devel gcc zlib-devel libpng12 # Fedora, RHEL, CentOS
git clone https://github.com/probonopd/AppImageKit.git
cd AppImageKit
cmake .
make
```

Once you have built AppImageKit, try making an AppImage, e.g., of Leafpad. The following has been tested on Ubuntu:

    export APP=leafpad && ./apt-appdir/apt-appdir $APP && ./AppImageAssistant $APP.AppDir $APP.AppImage && ./$APP.AppImage
    
(This is just a proof-of-concept, in reality you should use a proper "recipe" script or AppDirAssistant to create proper AppDirs - see an example at https://github.com/probonopd/AppImages/blob/master/recipes/subsurface.sh)

### Creating AppImages

The general workflow for creating an AppImage involves the following steps:
 1. __Gather suitable binaries__. If the application has already been compiled, you can use existing binaries (for example, contained in .tar.gz, deb, or rpm archives). Note that the binaries must not be compiled on newer distributions than the ones you are targeting. In other words, if you are targeting Ubuntu 9.10, you should not use binaries compiled on Ubuntu 10.04. For upstream projects, it might be advantageous to compile special builds for use in AppImages, although this is not required.
 2. __Gather suitable binaries of all dependencies__ that are not part of the base operating systems you are targeting. For example, if you are targeting Ubuntu, Fedora, and openSUSE, then you need to gather all libraries and other dependencies that your app requires to run that are not part of Ubuntu, Fedora, and openSUSE.
 3. __Create a working AppDir__ from your binaries. A working AppImage runs your app when you execute its AppRun file.
 4. __Turn your AppDir into an AppImage__. This compresses the contents of your AppDir into a single, self-mounting and self-executable file.
 5. __Test your AppImage__ on all base operating systems you are targeting. This is an important step which you should not skip. Subtle differences in distributions make this a must. While it is possible in most cases to create AppImages that run on various distributions, this does not come automatically, but requires careful hand-tuning.

While it would theoretically be possible to do all these steps by hand, AppImageKit contains tools that greatly simplify the tasks.

### Creating portable AppImages

For an AppImage to run on most systems, the following conditions need to be met:
 1. The AppImage needs to include all libraries and other dependencies that are not part of all of the base systems that the AppImage is intended to run on
 2. The binaries contained in the AppImage need to be compiled on a system not newer than the oldest base system that the AppImage is intended to run on
 3. The AppImage should actually be tested on the base systems that it is intended to run on

### Binaries compiled on old enough base system

The ingredients used in your AppImage should not be built on a more recent base system than the oldest base system your AppImage is intended to run on. Some core libaries, such as glibc, tend to break compatibility with older base systems quite frequently, which means that binaries will run on newer, but not on older base systems than the one the binaries were compiled on.

If you run into errors like this

    failed to initialize: /lib/tls/i686/cmov/libc.so.6: version `GLIBC_2.11' not found

then the binary is compiled on a newer system than the one you are trying to run it on. You should use a binary that has been compiled on an older system. Unfortunately, the complication is that distributions usually compile the latest versions of applications only on the latest systems, which means that you will have a hard time finding binaries of bleeding-edge softwares that run on older systems. A way around this is to compile dependencies yourself on a not too recent base system, and/or to use LibcWrapGenerator.

### Testing

To ensure that the AppImage runs on the intended base systems, it should be thoroughly tested on each of them. The following testing procedure is both efficient and effective: Get the previous version of Ubuntu, Fedora, and openSUSE Live CDs and test your AppImage there. Using the three largest distributions increases the chances that your AppImage will run on other distributions as well. Using the previous (current minus one) version ensures that your end users who might not have upgraded to the latest version yet can still run your AppImage. Using Live CDs has the advantage that unlike installed systems, you always have a system that is in a factory-fresh condition that can be easily reproduced. Most developers just test their software on their main working systems, which tend to be heavily customized through the installation of additional packages. By testing on Live CDs, you can be sure that end users will get the best experience possible.

I use ISOs of Live CDs, loop-mount them, chroot into them, and run the AppImage there. This way, I need approximately 700 MB per supported base system (distribution) and can easily upgrade to never versions by just exchanging one ISO file. The following script automates this for Ubuntu-like (Casper-based) and Fedora-like (Dract-based) Live ISOs:

    sudo ./AppImageAssistant.AppDir/testappimage /path/to/elementary-0.2-20110926.iso ./AppImageAssistant.AppImage

## Support

I support open source projects that wish to distribute their software as an AppImage. For closed source applications, I offer AppImage packaging and testing as a service.

## TODO

* Update http://www.portablelinuxapps.org/docs/1.0/ to remove all references to elficon; include changelog section
