# Tweak Development in Objective-C/Logos: ~~The Ultimate Guide~~ The WIP Guide

By Snoolie


# Part 1: Introduction

//explain some stuff, ex how theos is what we're be using but it's just a build system for our tweaks ex tools like dragonbuild exist and one can even make a tweak without using any build systems, logos being a pre-processor for objc, etc. this guide will also not only be creating demo tweaks, but also explain the process of how some actual tweaks work and re-goes through how they were made. also touch on some useful resources too.


# Part 2: Installing Theos

We're going to use the theos build system to build our tweaks. The first part of tweak development with theos, is, well, installing theos. Theos is helpful to us since it provides us with an easy way to build our tweak and provides us with Logos already. I personally use Theos on a Mac, but use is not that different from use on Linux and WSL (Windows Subsystem for Linux). Installation is pretty simple, and [explained on the official theos website here](https://theos.dev/docs/installation). I personally use a Mac, so all I need to do is install Homebrew and Xcode if I hadn't already, and run the command `bash -c "$(curl -fsSL https://raw.githubusercontent.com/theos/theos/master/bin/install-theos)"` in the terminal, but for Linux/Windows users installation is still fairly easy as well, just follow the instructions on the official theos website. If you do run into problems, though, feel free to ask in discords such as [the official theos discord](https://discord.gg/placeholder). Once we get that set up, we should be good to go.

# Part 3: Theos usage

After we've installed theos, I feel like I should explain some basic usage of it. Let's type echo $THEOS in our terminal - you should get out the directory where theos is. I'm going to demonstate the usage of NIC (New Instance Creator) which sets up a basic project for us. `cd` to a blank directory where you want to place your tweak at. Now, let's type `$THEOS/bin/nic.pl` in our terminal.

(Show image of nic.pl).

nic.pl (New Instance Creator) is a tool to auto set up a basic project for theos. It doesn't actually create the tweak, rather just sets up our tweak directory for us - you don't actually need nic.pl, you can just set it up yourself if you wish (which is actually what I do), but since a lot of people do use nic.pl I'll explain usage here. For now, we're obviously going to be using iphone/tool. You can type in the number next to it and enter to select that option.

nic.pl walks us through creating the tweak, Project Name will be the name of our tweak, I'm going to enter `DemoTweak`. For the package name, this is going to be the bundle identifier of our tweak - usually this follows the com.(authorname).(tweakname) format. Make sure that you don't make the bundle identifier the same as another package's - ex, if you set your bundle id to com.awesomedev.awesometweak and the user or you have a tweak called com.awesomedev.awesometweak, `dpkg` (how are tweaks are packaged - I'll touch on it more later) will think our tweak if the same as the other tweak, and installing it will act as an update/downgrade of that tweak and override it with ours - or the other way around with when the user has our tweak installed and installs the other tweak with the same bundle identifier, it will get rid of ours since it thinks the other tweak is an update/downgrade from our tweak. Also, don't use capital letters in our bundle id - this has been deprecated for a while. Once you decide on a package name / bundle id, it asks us for Author/Maintainer name, just input your name here. For MobileSubstrate Bundle Filter - this is asking the bundle id of the process that our tweak injects into. Our tweak we're making here is going to inject into SpringBoard, so we'll enter in com.apple.springboard. If we were making say a youtube tweak though, we'd want to enter in com.google.ios.youtube, since that's youtube's bundle id and we want to inject our tweak into youtube. Theos may also ask us for the processes to terminate upon initially installing the tweak, let's just enter and enter in nothing at the moment.

Congrats - you've set up a basic tweak. You can go to the directory in finder to see what NIC.pl set up. Before we get to making tweaks however, I'm going to continue going over basic theos usage, and go over what NIC.pl creates for us:

* `DemoTweak.plist - This is a plist file that specifies to the tweak injection system (ex Substrate/Substitute/libhooker/ElleKit) where to inject our tweak into. It'll have the bundle id of the process we specified we'd inject into (com.apple.springboard). `

* `control - This is our control file, specifying info about the tweak ex the name, bundle id, version etc, incredibly important as to how dpkg installs our tweak.`

* `Makefile - This is our makefile, it specifies for the make command how our tweak will be made.`

* `Tweak.xm / Tweak.x - This is going to be where the code for our tweak is.`

Now, I'm going to go into commands we'll be using regularly:

* `make clean` - Clean the build directory

* `make package` - Compile and build our tweak.

* `make package FINALPACKAGE=1` - This is like make package, but instead of compiling for debug, compiles with compiler optimizations and optimizes and does some more changes for a "clean" version of the tweak - use when making a tweak for public use.

* `make update-theos` - Updates theos

//remember to also mention basics of what to specify in the makefile, ex if we only want to compile for armv7 and to also specify that arm64e iOS 13 needs the Xcode 11 toolchain.


Now we've explained some theos usage - let's now get to making our first tweak.

# Part 4: Creating a tweak

This is the beginner, basic tweak that the majority of tweak development guides make - a basic hide dock tweak. However, most of those don't go into explaining how it works nor work on iPad due to only using SBDockView, so I'll try doing that. First, here's the mode we're going to put in our Tweak.xm/Tweak.x file:

```objc
#import <UIKit/UIKit.h>

@interface SBDockView : UIView
@end

@interface SBFloatingDockView : UIView
@end

@interface SBFloatingDockPlatterView : UIView
@end

%hook SBDockView
-(void)setHidden:(BOOL)arg0 {
  %orig(YES);
}
%end
%hook SBFloatingDockView
-(void)setHidden:(BOOL)arg0 {
  %orig(YES);
}
%end
%hook SBFloatingDockPlatterView
-(void)setHidden:(BOOL)arg0 {
  %orig(YES);
}
%end
```

So, what are we doing here?

Well, SBDockView, SBFloatingDockView, and SBFloatingDockPlatterView are all objects. An object can hold methods (sort of like functions), as well as properties and ivars. SBDockView, SBFloatingDockView, and SBFloatingDockPlatterView are all UIViews for the dock - a UIView is an object from the UIKit framework that represents, well, views - ex a programmer may want to have a view displayed on their app that contains an image, so they may use UIImageView (a UIView) to show the image in. Now, UIViews have a property called `hidden` - which, well, obviously tell if the view is hidden or not. Objective-C, for that hidden property, will have a getter and setter methods for interacting with that property - ex, SBDockView's setHidden:(BOOL0 property will set the property to the argument passed into it, ex setHidden:YES will set the hidden property to YES. The getter method, -(BOOL)hidden (which we do not need to hook as we are already hooking the setter), will return the value of the hidden property. SBDockView(et. all) inherits from UIView, so SBDockView and the rest has this hidden property, and these methods which set and return the value of it. Here, we have `@interface SBDockView : UIView` and etc for all other views we're hooking, which tells theos, hey, SBDockView inherits from UIView. Here's our SBDockView hook again:

```objc
%hook SBDockView
-(void)setHidden:(BOOL)arg0 {
  %orig(YES);
}
%end
```
What we're doing here is overriding SBDockView's method for setting the hidden property, setHidden:, with our custom code, that being `%orig(YES)`. %orig() is a Logos preprocessor directive, and using it in a void method will run the original code of the void method, ex, if we wanted to hook these but still have exactly the same behavior for whatever reason, we would have done `%orig(arg0)`, running the original method's code with it's original arg0 passed into it. Instead, though, we do %orig(YES) - this means that instead, we run the original method but always pass YES as an argument to it. In other words, forcing hidden to be true. SBDockView is what some iPhones use (ex I know notchless devices use this class), but (I think notched devices) and iPads use SBFloatingDockView, so we want to also hook that as well. SBFloatingDockPlatterView, I'm going to be honest, I forget what is used for, but I saw it in my source for AnotherLazyHideDockTweak, so I'm assuming it's used on some devices so we're hooking that as well. Now, `make package`, and our tweak's deb is made! Woo!

# Part 5: Into our deb

You'll notice it outputted a deb file. What is it? Well, let's just use a standard unarchiver tool to extract the contents of the .deb - you'll see `control`, `data`, and `debian-binary` (note: control/data may be .tar.xz / .tar.lzms, extract them again). The control contains our control file of our tweak, the data file though, outputs a folder. Depending on what version of theos you will be doing this on and the time it may be different, but you may get var > jb > usr > lib > TweakInject > (files) / Library > MobileSubstrate > DynamicLibraries (files). These files, you'll see a dylib, as well as the plist that we saw was created. Basically, the deb installs the `data` part to your device - ex you may have var/jb/usr/lib/TweakInject as to where your tweaks are stored or /usr/lib/TweakInject or /Library/MobileSubstrate/DynamicLibraries (just a symlink to formerly mentioned path iirc). The deb places those files there. Your dylib is your actual tweak, and the plist specifies the bundle id of the process to inject your dylib into.

//continue latr

# Part 6: Explaining APT/dpkg (+Creating a Repo)

# Part 7: Creating Pastcuts (1.0)

# Part 8: Coding in Objective-C

# Part 9: Creating Pastcuts (1.2)

# Part 10: UIKit Basics

# Part 11: Structure of the filesystem

//maybe split into multiple parts
//What's happening in our iPhone's terminal? - roughly touch the surface of binaries being in /usr/bin and /usr/local/bin and touch a little on sh

# Part 12: ObjC-API

# Part 13: Who needs Logos anyways?

//explain TweakWithoutLogos and TweakWithoutTheos

# Part 14: Finding Headers / Basic Reverse Engineering

# Part 15: What's a bootstrap?

//I'm prob never going to finish this lol

Reminders: Part 2 Also bring up theos for jailbroken iOS, Part 3 check if $THEOS env var is auto done
