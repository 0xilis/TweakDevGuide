# Tweak Development in Objective-C/Logos: ~~The Ultimate Guide~~ The WIP Guide

By Snoolie


# Part 1: Introduction

//explain some stuff, ex how theos is what we're be using but it's just a build system for our tweaks ex tools like dragonbuild exist and one can even make a tweak without using any build systems, logos being a pre-processor for objc, etc. this guide will also not only be creating demo tweaks, but also explain the process of how some actual tweaks work and re-goes through how they were made, providing helpful hands-on examples. also touch on some useful resources too.


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

Well, SBDockView, SBFloatingDockView, and SBFloatingDockPlatterView are all objects. An object can hold methods (sort of like functions), as well as properties and ivars. SBDockView, SBFloatingDockView, and SBFloatingDockPlatterView are all UIViews for the dock - a UIView is an object from the UIKit framework that represents, well, views - ex a programmer may want to have a view displayed on their app that contains an image, so they may use UIImageView (a UIView) to show the image in. Now, UIViews have a property called `hidden` - which, well, obviously tell if the view is hidden or not. Objective-C, for that hidden property, will have a getter and setter methods for interacting with that property - ex, SBDockView's setHidden:(BOOL) property will set the property to the argument passed into it, ex setHidden:YES will set the hidden property to YES. The getter method, -(BOOL)hidden (which we do not need to hook as we are already hooking the setter), will return the value of the hidden property. SBDockView(et. all) inherits from UIView, so SBDockView and the rest has this hidden property, and these methods which set and return the value of it. Here, we have `@interface SBDockView : UIView` and etc for all other views we're hooking, which tells theos, hey, SBDockView inherits from UIView. Here's our SBDockView hook again:

```objc
%hook SBDockView
-(void)setHidden:(BOOL)arg0 {
  %orig(YES);
}
%end
```
What we're doing here is overriding SBDockView's method for setting the hidden property, setHidden:, with our custom code, that being `%orig(YES)`. %orig() is a Logos preprocessor directive, and using it in a void method will run the original code of the void method, ex, if we wanted to hook these but still have exactly the same behavior for whatever reason, we would have done `%orig(arg0)`, running the original method's code with it's original arg0 passed into it. Instead, though, we do %orig(YES) - this means that instead, we run the original method but always pass YES as an argument to it. In other words, forcing hidden to be true. SBDockView is what some iPhones use (ex I know notchless devices use this class), but (I think notched devices) and iPads use SBFloatingDockView, so we want to also hook that as well. SBFloatingDockPlatterView, I'm going to be honest, I forget what is used for, but I saw it in my source for AnotherLazyHideDockTweak, so I'm assuming it's used on some devices so we're hooking that as well. Now, `make package`, and our tweak's deb is made! Woo!

That was a bit much admittedly, so here's some more examples of me trying to explain what we just did. Imagine this is code for SpringBoard itself so we can get a better understanding of what we're hooking:

```objc
SBDockView *dockView = [[SBDockView alloc]init];
/* SpringBoard has allocated the dock view */
/* now, we should set it as non-hidden! */
[dockView setHidden:NO];
/* that setHidden method means that the SBDockView will show! unless someone makes a tweak to change it to be YES, but it's not like that would happen! */
%end
```

Basically, when SpringBoard calls the setHidden method, our tweak forces it to be YES. This, in turn, hides the SBDockView.

# Part 5: Into our deb

You'll notice it outputted a deb file. What is it? Well, let's just use a standard unarchiver tool to extract the contents of the .deb - you'll see `control`, `data`, and `debian-binary` (note: control/data may be .tar.xz / .tar.lzms, extract them again). The control contains our control file of our tweak, the data file though, outputs a folder. Depending on what version of theos you will be doing this on and the time it may be different, but you may get var > jb > usr > lib > TweakInject > (files) / Library > MobileSubstrate > DynamicLibraries (files). These files, you'll see a dylib, as well as the plist that we saw was created. Basically, the deb installs the `data` part to your device - ex you may have var/jb/usr/lib/TweakInject as to where your tweaks are stored or /usr/lib/TweakInject or /Library/MobileSubstrate/DynamicLibraries (just a symlink to formerly mentioned path iirc). The deb places those files there. Your dylib is your actual tweak, and the plist specifies the bundle id of the process to inject your dylib into.

//continue latr

# Part 6: Explaining APT/dpkg (+Creating a Repo)

# Part 7: Creating Pastcuts (1.0)

Alright, time to touch on how one of my first tweaks was made, Pastcuts, a tweak to allow importing of iOS 15 shortcuts on iOS 13/14. I'll be pasting a lot of this from my previous [Pastcuts Writeup](https://github.com/0xilis/Pastcuts/blob/main/Writeup.md).

So we need to figure out what we need to hook beforehand. Back when shortcuts were shared as unsigned .shortcut files, I remember you could pull a shortcut file (which is just a plist) and modify it to have a lower WFWorkflowMinimumClientVersion, and boom import. Even if importing has been disabled on iOS 13/14 (excluding iOS 14 beta 1 where it was (possibly mistakenly?) re-enabled, and disabled the next beta), those unsigned shortcut files are still in use behind the scenes when importing shortcuts from iCloud in iOS 13/14 (you can even call the icloud API to receive the unsigned file if you want, ex links like these https://www.icloud.com/shortcuts/8d4e206d568d4aadb624b2a6191a3771 have this API https://www.icloud.com/shortcuts/api/records/8d4e206d568d4aadb624b2a6191a3771 in which contain a link to the signed shortcut for iOS 15+ and unsigned for iOS 14-), so that file must be in use somewhere.


So, what did I just say?


Well, there are two different types of plist formats shortcuts used for .shortcut files. Those are bplists and xml plists - I'm not touching bplists since everything should carry over to xml plists (at least for what we're doing). Rename an unsigned .shortcut file to .plist, and open in a plist editor. Here's an example of a xml plist .shortcut file, with just a comment action that says chocolate (excluding some keys that are not important and we aren't touching):


```xml
<key>WFWorkflowClientVersion</key>
<string>700</string>
<key>WFWorkflowClientRelease</key>
<string>2.0</string>
<key>WFWorkflowMinimumClientVersion</key>
<integer>411</integer>
<dict>
 <key>WFWorkflowActionIdentifier</key>
 <string>is.workflow.actions.comment</string>
 <key>WFWorkflowActionParameters</key>
 <dict>
  <key>WFCommentActionText</key>
  <string>Chocolate</string>
 </dict>
</dict>
```

So in a plist editor, if you change the WFWorkflowMinimumClientVersion in the file to something like 1, you can import it on an iOS 12 device without issues! (Well, that won't say the shortcut won't have issues - iOS 13 has a lot of actions that iOS 12 doesn't so the shortcut might not work if it uses one of those, but it will still allow importing).

So, how do we change that value automatically upon importing?


Shortcuts on iOS 13+ differs greatly behind the scenes from iOS 12-, with a lot of it being in iOS itself rather than the app - more specifically, ContentKit.framework, ActionKit.framework, and (the most important) WorkflowKit.framework. First let's understand what we need to hook. Thankfully shortcuts has a built-in action that makes this easy for us: View Content Graph of (x). Make a shortcut to get all shortcuts, choose list action, then view content graph. Run and choose a shortcut, and View Content graph should appear. Tap on the shortcut's name, then Shortcut. We can see 5 items - WFWorkflowReference, Shortcut, WFImage, WFWorkflowRecord, and NSString. We're going to modify WFWorkflowRecord. Let's take a look at the header - https://headers.cynder.me/index.php?sdk=ios/13.7&fw=/PrivateFrameworks/WorkflowKit.framework&file=%2FHeaders%2FWFWorkflowRecord.h. Oooo, NSString *minimumClientVersion - that sounds useful! Just hook minimumClientVersion in WFWorkflowRecord, and make it return a NSString of 1 (@"1"). Boom - You just created Pastcuts 1.0!


Well, this actually honestly wasn't how I created it. Remember how I said I was a dumb kid doing random Flex 3? Yeah, admittedly that's just what I did: edit random Flex 3 stuff until I saw it worked. This is why older Pastcuts versions not only hook WFWorkflowRecord minimumClientVersion (the one thing that makes Pastcuts 1.0 word), but also WFWorkflowReference minimumClientVersion, which is unneeded, as well as some other random stuff that had methods that sounded related to importing with as isSupportedOnThisDevice, which isn't related to importing at *all* and is completely unneeded.

I really recommend learning some basic RE when making tweaks rather than just hooking random headers and hoping for the best. Chapter 14 of this book / Part 14 of this PDF will go over some basics. However, playing around and hooking random headers when you start *can* be a good way to get more familiar with tweaks, as long as you of course don't accidentally hook something dangerous (ex, if you see a method called "deleteAllData", then maybe that isn't a good idea to hook...).

Here's all that's needed to recreate Pastcuts 1.0:

```objc
%hook WFWorkflowRecord
-(id)minimumClientVersion {
  return @"1";
}
%end
```
(Inside of `Tweak.xm`/`Tweak.x`.)

Now, make our plist affect com.apple.shortcuts (Shortcuts's bundle id) as well as com.apple.WorkflowKit (WorkflowKit's bundle id).
```plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Filter</key>
	<dict>
		<key>Bundles</key>
		<array>
			<string>com.apple.WorkflowKit</string>
			<string>com.apple.shortcuts</string>
		</array>
	</dict>
</dict>
</plist>
```
(Inside of `Pastcuts.plist`.)

Okay, so, what's wrong here? Well, in 1.2 I wanted Pastcuts to convert shortcut actions when importing. I realized that my logic was bad. Why? Well, we're hooking minimumClientVersion in EVERY shortcut loaded. This might be bad for performance / battery, and while we're not doing anything that can go wrong that much, once we get to hooking actions, if we make one mistake, not just that shortcut gets affected but EVERY shortcut gets affected, so it's pretty dangerous as well.

I still wasn't that experienced with RE, but I had improved by Obj-C skills a bit by this point - instead of RE, I instead just plopped in a NSLog, looked at console, and saw how many times it was called and was like, "okay, this definitely is called too much". I still didn't know RE so I just messed with random classes Flex 3 showed, and found WFSharedShortcut. After trying an NSLog with a hook *this* time, I saw that it seems to not be called when loading shortcuts, only when importing. So I decided that WFSharedShortcut was a much better option for this than WFWorkflowRecord. Current Pastcuts, 1.3.0, still actually uses WFSharedShortcut, fun fact, and some other tweaks of mine such as Safecuts, which mitigates an iOS 15.0-15.3.1 hidden action vuln also use it (although I should really switch to using WFShortcutExporter for Safecuts - it's brand new to iOS 15, and if you're doing iOS 15, it's more better suited for this type of stuff). I would still recommend doing, uh, good RE and understanding exactly how WFSharedShortcut works, but at the time this was all I knew and I guess it's better than nothing.


So, let's just switch to hooking (also in WorkflowKit) WFSharedShortcut's workflowRecord instead. Let's do `WFWorkflowRecord *workflowRecord = %orig;`, %orig will return what the getter method would have originall returned (aka the unmodified WFWorkflowRecord). Then, `[workflowRecord setMinimumClientVersion:@"1"];` to set minimumClientVersion to be a NSString being @"1". Then just return workflowRecord (our modified WFWorkflowRecord), and boom, now we only hook when a shortcut is imported, making us MUCH more optimized! I was planning on waiting for Pastcuts 1.2, but I decided this optimization is enough to deserve a quick 1.1.2 release.

```objc
%hook WFSharedShortcut
-(WFWorkflowRecord *)workflowRecord {
  WFWorkflowRecord *workflowRecord = %orig;
  /* workflowRecord is what this getter would have returned originally */
  /* so, lets set minimumClientVersion to 1 */
  [workflowRecord setMinimumClientVersion:@"1"];
  /* and then, returned our modified workflowRecord */
  return workflowRecord;
}
%end
```

# Part 8: Coding in Objective-C

Here, a simple Hello World program in C:

```c
#include <stdio.h>

int main(void) {
 printf("Hello World\n");
 return 0;
}
```

Here's that, but using NSLog instead from Objective C:

```c
#include <Foundation/Foundation.h>

int main(void) {
 NSLog(@"Hello World");
 return 0;
}
```

What are we passing into the NSLog function, `@"Hello World"` is a NSString object containing the text "Hello World". Now, here's a custom class:

```objc
@interface OurOwnClass : NSObject {
 int ourIvar; //an instance variable
}
@property(nonatomic, readwrite) ourProperty;
-(void)anInstanceMethod;
+(void)aStaticMethod;
@end
@implementation OurOwnClass
-(void)anInstanceMethod {
 if (ourIvar > 5) {
  ourIvar += 1;
  NSLog(@"Added 1 to ourIvar");
 } else {
  NSLog(@"ourIvar has not been initialized");
 }
}
+(void)aStaticMethod {
 NSLog(@"This is our static method!");
}
@end
```

Here's us using the class in a program:

```objc
#include <Foundation/Foundation.h>

@interface OurOwnClass : NSObject {
 int ourIvar; //an instance variable
}
@property(nonatomic, readwrite) int ourProperty;
-(void)anInstanceMethod;
+(void)aStaticMethod;
@end
@implementation OurOwnClass
-(void)anInstanceMethod {
 if ([self ourProperty] == 5) {
  ourIvar = 8;
 }
 if (ourIvar > 5) {
  ourIvar += 1;
  NSLog(@"Added 1 to ourIvar");
 } else {
  NSLog(@"ourIvar has not been initialized or is not greater than 5");
 }
}
+(void)aStaticMethod {
 NSLog(@"This is our static method!");
}
@end

int main(void) {
 OurOwnClass *thisIsAnObject = [[OurOwnClass alloc]init];
 [thisIsAnObject anInstanceMethod]; //this will output ourIvar has not been initialized or is not greater than 5
 [thisIsAnObject setOurProperty:5]; //setting ourProperty to 5
 [thisIsAnObject anInstanceMethod]; //since ourProperty is now 5, ourIvar will be set to 8, and it is greater than 5 so 1 will be added to ourIvar
 NSLog(@"%d",[thisIsAnObject ourProperty]); //this will output 5, the current value of ourProperty
 [thisIsAnObject setOurProperty:2]; //setting ourProperty to 2
 [thisIsAnObject anInstanceMethod]; //ourProperty is no longer 5, so ourIvar won't be set to 8. however ourIvar is 6, so one will be added to it.
 [OurOwnClass aStaticMethod]; //outputs This is our static method!
 return 0;
}
```

Lemme explain:

So, we have a class, OurOwnClass. Classes aren't objects, but rather they define what our objects do. We are creating a new object, thisIsAnObject, that's type is OurOwnClass. In other words, we're creating an instance of our OurOwnClass class, and thisIsAnObject is a pointer to that.

So, an instance variable, or ivar for short, are variables for our object thisIsAnObject, but are private to it - aka every object we make from OurOwnClass will have its own ourIvar.

You'll see @property(nonatomic, readwrite) int ourProperty - this will autogen for us a setter (setOurProperty:), getter (ourProperty), and ivar (\_ourProperty) for ourProperty.

What's a setter/getter? Well, we may want to access the ivar's value and change it. Setters/getters do exactly that. A getter will be autogen'd for us if we don't declare our custom one. We aren't declaring this property as readonly, so a setter will also be autogen'd for us if we don't declare our custom one. Here's the same thing, but instead of @property(nonatomic, readwrite) int ourProperty do the work for us, we're using our own getter/setter for our ivar:

```objc
#include <Foundation/Foundation.h>

@interface OurOwnClass : NSObject {
 int ourIvar; //an instance variable
 int ourProperty;
}
-(void)anInstanceMethod;
+(void)aStaticMethod;
@end
@implementation OurOwnClass
-(void)anInstanceMethod {
 if ([self ourProperty] == 5) {
  ourIvar = 8;
 }
 if (ourIvar > 5) {
  ourIvar += 1;
  NSLog(@"Added 1 to ourIvar");
 } else {
  NSLog(@"ourIvar has not been initialized or is not greater than 5");
 }
}
-(int)ourProperty {
 return ourProperty;
}
-(void)setOurProperty:(int)arg0 {
 ourProperty = arg0;
}
+(void)aStaticMethod {
 NSLog(@"This is our static method!");
}
@end

int main(void) {
 OurOwnClass *thisIsAnObject = [[OurOwnClass alloc]init];
 [thisIsAnObject anInstanceMethod]; //this will output ourIvar has not been initialized or is not greater than 5
 [thisIsAnObject setOurProperty:5]; //setting ourProperty to 5
 [thisIsAnObject anInstanceMethod]; //since ourProperty is now 5, ourIvar will be set to 8, and it is greater than 5 so 1 will be added to ourIvar
 NSLog(@"%d",[thisIsAnObject ourProperty]); //this will output 5, the current value of ourProperty
 [thisIsAnObject setOurProperty:2]; //setting ourProperty to 2
 [thisIsAnObject anInstanceMethod]; //ourProperty is no longer 5, so ourIvar won't be set to 8. however ourIvar is 6, so one will be added to it.
 [OurOwnClass aStaticMethod]; //outputs This is our static method!
 return 0;
}
```

This is... not really covering much. At all. Objecctive-C is an entire programming language and it's not like I can explain it all in a singular part of a tweak dev guide. For learning, I recommend other resourves specifically designed to teach ObjC, such as [Learn Objective-C in 24 Days](https://github.com/uroboro/Learn-Objective-C-in-24-Days-Clone). Maybe I might expand this section a bit to explain a bit more but I'll likely never get to a good full rundown to teach you the language, as sadly I do not have the time nor the patience to type an entire ObjC guide here, however there are a *lot* of resources to learn, like the resource I listed here.

//finish latr

# Part 9: Creating SnoolieBattery

SnoolieBattery is a tweak that, much like all other battery percentage tweaks, 

# Part 10: Creating Pastcuts (1.2)

Okay, so we can import all Shortcuts imported now. I missed that gallery shortcuts won't be under WFSharedShortcut, so we should also do the same hook for WFGalleryShortcut's workflowRecord as well. Now, even though we can import any action, iOS 13/14 doesn't have every action iOS 15 does, does it? Let's fix that.

So first off I want to thank u/gluebyte for documenting some of the backwards (in)compatibility of iOS 15. See the post here https://www.reddit.com/r/shortcuts/comments/opak23/backward_incompatibility_of_ios_15_shortcuts/. Let's start by converting the stop action back to the exit action.

We need to get the actions, obv. Here's WFSharedShortcut's header, which you can find here: [https://headers.cynder.me/index.php?sdk=ios/13.5&fw=/PrivateFrameworks/WorkflowKit.framework&file=Headers%2FWFSharedShortcut.h](https://headers.cynder.me/index.php?sdk=ios/13.5&fw=/PrivateFrameworks/WorkflowKit.framework&file=Headers%2FWFSharedShortcut.h):

```objc
@interface WFSharedShortcut : NSObject <WFCloudKitItem, WFLoggableObject>



@property (readonly, nonatomic) NSDate *createdAt; // ivar: _createdAt
@property (readonly, nonatomic) NSString *createdBy; // ivar: _createdBy
@property (readonly, copy) NSString *debugDescription;
@property (readonly, copy) NSString *description;
@property (readonly) NSUInteger hash;
@property (retain, nonatomic) WFWorkflowIcon *icon;
@property (retain, nonatomic) NSNumber *iconColor; // ivar: _iconColor
@property (retain, nonatomic) WFFileRepresentation *iconFile; // ivar: _iconFile
@property (retain, nonatomic) NSNumber *iconGlyph; // ivar: _iconGlyph
@property (readonly, nonatomic) CKRecordID *identifier; // ivar: _identifier
@property (copy, nonatomic) NSString *name; // ivar: _name
@property (readonly, nonatomic) NSDictionary *propertiesForEventLogging;
@property (retain, nonatomic) WFFileRepresentation *shortcutFile; // ivar: _shortcutFile
@property (readonly) Class superclass;
@property (retain, nonatomic) WFWorkflowRecord *workflowRecord; // ivar: _workflowRecord


+(id)properties;
+(id)recordType;
-(id)sharingURL;
-(void)ensureFileAssets;
-(void)setCreatedAt:(id)arg0 modifiedAt:(id)arg1 createdBy:(id)arg2 ;


@end
```

The workflowRecord property is obviously going to hold a WFWorkflowRecord for the shortcut we're importing. Here is WFWorkflowRecord's header:

```ojbc
@interface WFWorkflowRecord : WFRecord <WFNaming>



@property (copy, nonatomic) NSSet *accessResourcePerWorkflowStates; // ivar: _accessResourcePerWorkflowStates
@property (copy, nonatomic) NSArray *actions; // ivar: _actions
@property (copy, nonatomic) NSString *actionsDescription; // ivar: _actionsDescription
@property (copy, nonatomic) NSString *associatedAppBundleIdentifier; // ivar: _associatedAppBundleIdentifier
@property (nonatomic) NSUInteger cachedSyncHash; // ivar: _cachedSyncHash
@property (readonly, nonatomic) BOOL conflictOfOtherWorkflow; // ivar: _conflictOfOtherWorkflow
@property (retain, nonatomic) NSDate *creationDate; // ivar: _creationDate
@property (readonly, nonatomic) NSUInteger estimatedSize; // ivar: _estimatedSize
@property (copy, nonatomic) NSString *galleryIdentifier; // ivar: _galleryIdentifier
@property (nonatomic) BOOL hiddenFromLibraryAndSync; // ivar: _hiddenFromLibraryAndSync
@property (nonatomic) BOOL hiddenInComplication; // ivar: _hiddenInComplication
@property (retain, nonatomic) WFWorkflowIcon *icon; // ivar: _icon
@property (copy, nonatomic) NSArray *importQuestions; // ivar: _importQuestions
@property (copy, nonatomic) NSArray *inputClasses; // ivar: _inputClasses
@property (readonly, nonatomic) BOOL isDeleted; // ivar: _isDeleted
@property (copy, nonatomic) NSString *lastMigratedClientVersion; // ivar: _lastMigratedClientVersion
@property (copy, nonatomic) NSString *lastSavedOnDeviceName; // ivar: _lastSavedOnDeviceName
@property (nonatomic) NSInteger lastSyncedHash; // ivar: _lastSyncedHash
@property (copy, nonatomic) NSString *legacyName; // ivar: _legacyName
@property (readonly, nonatomic) NSNumber *location; // ivar: _location
@property (copy, nonatomic) NSString *minimumClientVersion; // ivar: _minimumClientVersion
@property (retain, nonatomic) NSDate *modificationDate; // ivar: _modificationDate
@property (copy, nonatomic) NSString *name; // ivar: _name
@property (retain, nonatomic) WFWorkflowQuarantine *quarantine; // ivar: _quarantine
@property (nonatomic) NSInteger remoteQuarantineStatus; // ivar: _remoteQuarantineStatus
@property (copy, nonatomic) NSString *source; // ivar: _source
@property (readonly, copy, nonatomic) NSString *wfName;
@property (copy, nonatomic) NSString *workflowSubtitle; // ivar: _workflowSubtitle
@property (copy, nonatomic) NSArray *workflowTypes; // ivar: _workflowTypes


+(id)defaultPropertyValues;
-(BOOL)hiddenFromLibraryAndSync;
-(BOOL)hiddenInComplication;
-(BOOL)isDeleted;
-(BOOL)isEquivalentForSyncTo:(id)arg0 ;
-(BOOL)saveChangesToStorage:(id)arg0 error:(*id)arg1 ;
-(NSUInteger)syncHash;
-(id)fileRepresentation;


@end
```
So it seems that actions is the property that contains our actions as an NSArray. `NSArray *origShortcutActions = (NSArray *)[rettype actions];`. I also create a mutable copy of the actions as newMutableShortcutActions so we can modify it. `NSMutableArray *newMutableShortcutActions = [origShortcutActions mutableCopy];` Then I proceed to have a for loop to loop through all the actions in origShortcutActions. Actions are NSDictionaries. A dictionary is basically an object that holds other objects as values, with keys for them - what I mean is, here I'm creating a NSDictionary containing two keys, Key1 and Key2, with Key1 being a NSString holding Value1, and Key2 being a NSString holding Value2: `NSDictionary *aDict = [[NSDictionary alloc]initWithObjectsAndKeys:@"Value2",@"Key2",@"Value1",@"Key1",nil];`. We can use `objectForKey:` to retrive the object for the key, ex, `NSLog(@"%@",[aDict objectForKey:@"Key2"])` will output Value2, the NSString value for the key Key2.

The action NSDictionary holds some keys representing values for the actions. `WFWorkflowActionIdentifier` is a string representing the action'd id. To check if the action is a stop action, we just check if the action in the loop's `WFWorkflowActionIdentifier` is equal to string is.workflow.actions.output.

If so, I create a mutable copy of the action to modify it. Then I change the `WFWorkflowActionIdentifier` to equal is.workflow.actions.exit. I then proceed to attempt to get its output and correspond to is.workflow.actions.exit's way. You don't need to understand how is.workflow.actions.exit works, what matters here in this guide is that you understand what objective-c code does.

Here's an example of this:

TODO: The code shown here is absolutely horrible, remember to rewrite it...

```objc
%hook WFSharedShortcut
-(id)workflowRecord {
  id workflowRecord = %orig; //the original wfworkflowrecord that would have returned
  [workflowRecord setMinimumClientVersion:@"1"];
  NSArray *origShortcutActions = (NSArray *)[workflowRecord actions];
  NSMutableArray *newMutableShortcutActions = [origShortcutActions mutableCopy]; //what is going to be our new actions
  int shortcutActionsObjectIndex = 0;
  for (id shortcutActionsObject in origShortcutActions) {
    //this safety check is only needed if you are unaware of in actions it potentially contains more than NSDictionaries.
    if ([shortcutActionsObject isKindOfClass:[NSDictionary class]]){
      if ([[shortcutActionsObject objectForKey:@"WFWorkflowActionIdentifier"] isEqualToString:@"is.workflow.actions.output"]) {
        NSMutableDictionary *mutableShortcutActionsObject = [shortcutActionsObject mutableCopy];

        [mutableShortcutActionsObject setObject:@"is.workflow.actions.exit" forKey:@"WFWorkflowActionIdentifier"];
        if ([[[[mutableShortcutActionsObject objectForKey:@"WFWorkflowActionParameters"] objectForKey:@"WFOutput"] objectForKey:@"Value"] objectForKey:@"attachmentsByRange"] objectForKey:@"{0, 1}"]) {
    //in iOS 15, if an Exit action has output it's converted into the Output action, so we convert it back

          NSDictionary *actionParametersWFResult = [[NSDictionary alloc] initWithObjectsAndKeys:@"placeholder", @"Value", @"WFTextTokenAttachment", @"WFSerializationType", nil];
          NSMutableDictionary *mutableActionParametersWFResult = [actionParametersWFResult mutableCopy];
          [mutableActionParametersWFResult setObject:[[[[[mutableShortcutActionsObject objectForKey:@"WFWorkflowActionParameters"] objectForKey:@"WFOutput"] objectForKey:@"Value"] objectForKey:@"attachmentsByRange"] objectForKey:@"{0, 1}"] forKey:@"Value"];
          NSDictionary *actionParameters = [[NSDictionary alloc] initWithObjectsAndKeys:@"placeholder", @"WFResult", nil];
          NSMutableDictionary *mutableActionParameters = [actionParameters mutableCopy];
          [mutableActionParameters setObject:mutableActionParametersWFResult forKey:@"WFResult"];
          [mutableShortcutActionsObject setObject:mutableActionParameters forKey:@"WFWorkflowActionParameters"];
        }
        newMutableShortcutActions[shortcutActionsObjectIndex] = [[NSDictionary alloc] initWithDictionary:mutableShortcutActionsObject];
      }
    }
  }
  //newMutableShortcutActions holds our new modified actions now
  [rettype setActions:newMutableShortcutActions]; //setting the wfworkflowrecord's actions to be ours
  return rettype; //returning our modified wfworkflowrecord
}
%end
```

# Part 11: More with Logos

Let's say we want our tweak to run some code when starting up. %ctor and %dtor are ways to do that - they're similar, but the difference is that %ctor executes after the binary loads, and %dtor executes before. We also might want to group some of our hooks - %group is a way to do that.

```
%group MyGroupOfHookOrHooks
%hook SomeClass
-(void)aMethod {
 NSLog(@"This method will only be hooked when anInt is 3!");
}
%end
%end
%hook AnotherClass
-(void)aMethod {
 NSLog(@"The hook for this method is ungrouped.");
}
%end
%ctor {
 NSLog(@"After binary loaded.");
 int anInt = 2;
 if (anInt == 3) { //anInt is 2, but if it were 3, the hooks in MyGroupOfHookOrHooks would be initialized.
  %init(MyGroupOfHookOrHooks);
 }
 %init(_ungrouped); //initialize all ungrouped hooks
}
%dtor {
 NSLog(@"Before binary loaded.");
}
```

An example of when you're going to use constructors is say, if your tweak has preferences and you don't want something to be hooked if your user doesn't set something in their preferences.

Want to add a method to an object? Use `%new`. Here I'm adding `customMethod` to AwesomeObject:

```objc
%hook AwesomeObject
%new
-(void)customMethod {
 NSLog(@"The custom method of this awesome object has been called.");
}
%end
```

So we've been hooking Objective-C methods for all this time, but what if we need to hook a function? `%hookf` is for that. Let's imagine the process we're hooking into has a function called `int veryBoringFunction(char *aString)`. To make it always return 5 we can:

```objc
int veryBoringFunction(char *aString);

%hookf(int, veryBoringFunction, char *aString) {
  return 5;
}
```

# Part 12: Creating Unsigncuts

We will now be recreating Unsigncuts, an Objective-C tweak that allows unsigned shortcut files to be supported. If you don't know what that is, don't worry - STILL listen, because this chapter is important, as it will be your first introduction to using a dissassembler for RE, which will be incredibly helpful for understanding how stuff works behind the scenes, and is a must-have for tweak dev.

So, as I explained in the Creating Pastcuts tutorial, one of the backbones to Shortcuts is `WorkflowKit.framework`. When we import an unsigned shortcut on iOS 15+, you'll get this warning: "Importing unsigned shortcut files is not supported. Please use another sharing option.". But what *could* be this, exactly? We're going to use a dissassembler to find out. I'm going to be using hopper - don't worry about paying, I'm using the trial version as well, lol.

So, take the WorkflowKit binary from `/System/Library/PrivateFrameworks/WorkflowKit.framework`. If you don't see it, it's in dyld shared cache. Extract the dyld shared cache (I'll provide an explanation for how to do so later) actually since I'm lazy just go to [https://headers.cynder.me](https://headers.cynder.me) and download the binary. Open Hopper disassembler, and drag and drop it into Hopper.

[Image]

Tap on "Str" for strings in the binary. Now, let's search: "Importing unsigned shortcut files " and we get the string. Tap on it to go to it.

[Image]

Scroll to the right, and find `DATA XREF=cfstring_Importing_unsigned_shortcut_files...`. DATA XREF= is where this string is referenced.

[Image]

Double click on `cfstring_Importing_unsigned_shortcut_files_is_not_supported__Please_use_another_sharing_option_`.

[Image]

This takes us to a new location in the binary. Now, scroll to the right again, and you'll see `DATA XREF=+[WFSharingSettings shortcutFileSharingDisabledError]+64`. Jackpot! This string seems to be associated with this error method. Now, on the section with Labels, Proc, Str, the star and dot, tap back on `Labels` and search for `shortcutFileSh`. You'll get three labels all named `aShortcutfilesh`... just tap on one until you get to `shortcutFileSharingDisabledError`. Scroll to the right again - I see three things this is used in: `DATA XREF=+[WFSharingSettings shortcutFileSharingDisabledAlert]+24, -[WFShortcutExtractor extractWorkflowFile:completion:]+304, 0x5c69d8`. Woah, `-[WFShortcutExtractor extractWorkflowFile:completion:]` - that sounds interesting and it could be related to importing. Lets double click on that.

[Image]

Ok, so up until this point, you have had a top layer with some options, one of the top layer buttons says `mov add` - this means you're seeing the assembly of this method. However, Hopper has a pretty decent Objective-C decompiler that we can use. We don't need to worry about knowing assembly right now, the main purpose of this section is to introduce you to using disassembler tools, not assembly, so we're going to use the Objective-C decompiler. Tap on the button that says `if(b) f(x);`.

[Image]

We now have an accurate depiction of the function's code, in a (somewhat) understandable fashion to us! Yippe! Let's look at this snippet of code from this function:

```objc
//...
r21 = arg0;
//...
//the real meat is below vvvv
if ((VCIsInternalBuild() != 0x0) && ([WFSharingSettings shortcutFileSharingEnabled] != 0x0)) {
            r0 = [r21 suggestedName];
            r29 = r29;
            r0 = [r0 retain];
            r22 = r0;
            if (r0 != 0x0) {
                    [r21 extractWorkflowFile:r19 shortcutName:r22 shortcutFileContentType:0x0 iCloudIdentifier:0x0 completion:r20];
            }
            else {
                    r23 = [[r19 wfName] retain];
                    [r21 extractWorkflowFile:r19 shortcutName:r23 shortcutFileContentType:0x0 iCloudIdentifier:0x0 completion:r20];
                    [r23 release];
            }
    }
    else {
            if (([r21 allowsOldFormatFile] & 0x1) != 0x0) {
                    r0 = [r21 suggestedName];
                    r29 = r29;
                    r0 = [r0 retain];
                    r22 = r0;
                    if (r0 != 0x0) {
                            [r21 extractWorkflowFile:r19 shortcutName:r22 shortcutFileContentType:0x0 iCloudIdentifier:0x0 completion:r20];
                    }
                    else {
                            r23 = [[r19 wfName] retain];
                            [r21 extractWorkflowFile:r19 shortcutName:r23 shortcutFileContentType:0x0 iCloudIdentifier:0x0 completion:r20];
                            [r23 release];
                    }
            }
            else {
                    r22 = [[WFSharingSettings shortcutFileSharingDisabledError] retain];
                    (*(r20 + 0x10))(r20, 0x0, r22);
            }
    }
```

Ok, so, it seems like the ability to import unsigned shortcut files is still there, but just locked away by this function. This function checks if `[WFSharingSettings shortcutFileSharingEnabled]` returns true (the old way to enable shortcut file importing on iOS 14-), however also needs VCIsInternalBuild() function from VoiceShortcutsClient.framework to return true as well. If this isn't the case - [r21 allowsOldFormatFile]. r21, as we can see be the above `r21 = arg0`, is `arg0` according to Hopper's Objective-C decompiler. However, this arg0 isn't actually the args passed into the method, rather instead, it's actually `self` instead. So `if (([r21 allowsOldFormatFile] & 0x1) != 0x0) {` is actually `if ([self allowsOldFormatFile]) {`. Normally, this is false, and [WFSharingSettings shortcutFileSharingDisabledError] will be called. However, let's look at our headers:

```objc
@interface WFShortcutExtractor : NSObject
@property (readonly, nonatomic) BOOL allowsOldFormatFile;
@property (readonly, nonatomic) WFFileRepresentation *extractingFile;
@property (readonly, nonatomic) NSURL *extractingURL;
@property (readonly, nonatomic) NSInteger fileAdoptionOptions;
@property (readonly, nonatomic) BOOL skipsMaliciousScanning;
@property (readonly, copy, nonatomic) NSString *sourceApplication;
@property (readonly, copy, nonatomic) NSString *suggestedName;
+(BOOL)isShortcutFileType:(WFFileType *)fileType;
-(BOOL)allowsOldFormatFile;
-(BOOL)skipsMaliciousScanning;
-(id)initWithFile:(WFFileRepresentation *)file allowsOldFormatFile:(BOOL)allowOldFileFormat skipsMaliciousScanning:(BOOL)skipScanning suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(id)initWithFile:(WFFileRepresentation *)file allowsOldFormatFile:(BOOL)allowOldFileFormat suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(id)initWithFile:(WFFileRepresentation *)file suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(id)initWithURL:(NSURL *)url allowsOldFormatFile:(BOOL)allowOldFileFormat fileAdoptionOptions:(NSInteger)options suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(id)initWithURL:(NSURL *)url allowsOldFormatFile:(BOOL)allowOldFileFormat skipsMaliciousScanning:(BOOL)skipScanning fileAdoptionOptions:(NSInteger)options suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(id)initWithURL:(NSURL *)url allowsOldFormatFile:(BOOL)allowOldFileFormat skipsMaliciousScanning:(BOOL)skipScanning suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(id)initWithURL:(NSURL *)url allowsOldFormatFile:(BOOL)allowOldFileFormat suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(id)initWithURL:(NSURL *)url fileAdoptionOptions:(NSInteger)options suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(id)initWithURL:(NSURL *)url suggestedName:(NSString *)suggestName sourceApplication:(NSString *)app;
-(void)extractRemoteShortcutFileAtURL:(id)shortcutFileURL completion:(id)completion;
-(void)extractShortcutFile:(WFFileRepresentation*)shortcutFile completion:(id)completion;
-(void)extractShortcutWithCompletion:(id)completion;
-(void)extractSignedShortcutFile:(WFFileRepresentation*)shortcutFile allowsRetryIfExpired:(BOOL)allowRetry completion:(id)completion;
-(void)extractSignedShortcutFile:(WFFileRepresentation*)shortcutFile completion:(id)completion;
-(void)extractWorkflowFile:(WFFileRepresentation*)shortcutFile completion:(id)completion;
-(void)extractWorkflowFile:(WFFileRepresentation*)shortcutFile shortcutName:(NSString *)name shortcutFileContentType:(NSInteger)shortcutType iCloudIdentifier:(id)shortcutIdentifier completion:(id)completion;
@end
```

So in other words, `allowsOldFormatFile` is just a BOOL property of WFShortcutExtractor. If we hook it, this function will allow us to import unsigned shortcut files~! Yippie!

It's pretty straightforward from here, you can probably guess what to do at this point.

```objc
%hook WFShortcutExtractor
-(BOOL)allowsOldFormatFile {
 return YES;
}
%end
```

This was our first guide to using a disassembler! Yay! Now you won't have to just guess what methods do what now, RE will allow you to figure out what some stuff does with much less trial and error, insanely a must-have for tweak-dev. You will be applying this skill a lot, and I mean a lot.

# Part 13: UIKit Basics

//cover some basics of uikit here, show examples of gesture recognizers, adding subviews to uiviews, and presenting view controllers.  not gonna explain everything obv and explain why learning here prob won't do you much good but link to good resources

# Part 14: Structure of the filesystem

Let's get Filza or Santander, and go to our root directory, `/`.

/var, which is a symlink to /private/var, is the userFS. Most things out of var are in the system partition.

Some important directories for SystemFS:

`/Applications` - System apps are stored here.

`/usr/bin` and `/usr/local/bin` - holds core system binaries.

`/usr/lib/TweakInject` - where dylibs and plists for our tweaks are stored are stored.

//finish later

//maybe split into multiple parts
//What's happening in our iPhone's terminal? - roughly touch the surface of binaries being in /usr/bin and /usr/local/bin and touch a little on sh

# Part 15: Substrate APIs and ObjC-API

Ever wondered how Objective-C works behind the scenes?

//finish latr

# Part 16: Who needs Logos anyways?

//explain TweakWithoutLogos and TweakWithoutTheos

# Part 17: Finding Headers / Basic Reverse Engineering

# Part 18: What's a bootstrap?

//I'm prob never going to finish this lol

Reminders: Part 2 Also bring up theos for jailbroken iOS, Part 3 check if $THEOS env var is auto done
