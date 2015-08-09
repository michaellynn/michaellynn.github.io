---
summary: "pyobjc is awesome, but sometimes it needs a little help"
layout: post
tags: python osx hopper
date: "2015-08-08 16:15:00"
title: Learn you a better pyobjc Bridgesupport signature 
---

This blog post is about correcting function signatures in pyobjc.

This blog post has a backstory.

Recently I received a challenge that was posed by [Shea Craig](https://twitter.com/shea_craig) to me on the [Macadmins Slack](http://macadmins.org) in the **#autopkg** channel:

> @sheagcraig: Also, it's possible that we could try to do a mount via PyObjC stuff (paging @frogor)

_<small>*(frogor is one of my [many names](/about))</small>_

In the discussion, a few people were wondering if it was possible to mount network shares with python in OS X (_of course it is_) because using the command-line ```mount``` was running into some odd situations with Kerberos ticket handling.

You could call out to AppleScript (```osascript```) to do it with "mount volume", but the rest of the code involved was already written in python and there was interest in reducing the number of external tools used rather than replacing one with another.

This sounded like a fun challenge!

So the first thing I did was pop open the relevant AppleScript code in [Hopper](http://hopperapp.com). In this instance, that particular code is located in the bundle: /System/Library/ScriptingAdditions/StandardAdditions.osax

How did I find that, you might ask?

Unless you're using AppleScript to speak to a particular application that has its own custom commands (aka dictionary), the built-in commands for AppleScript that aren't just part of the AppleScript language itself are mostly stored in the "Standard Additions" scripting addition.

If you open up Script Editor and select _File -> Open Dictionary..._ and select StandardAdditions.osax, you can see the "mount volume" command and its associated documentation.

![](</images/2015-08-08-learn-you-a-better-pyobjc-bridgesupport-signature/StandardAdditions.png>)

Opening the executable in Hopper, unfortunately you can't just search for "mount volume" and find the relevant code. This is because AppleScript additions have a translation layer called a [Scripting Definition File](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ScriptableCocoaApplications/SApps_creating_sdef/SAppsCreateSdef.html) that maps the AppleScript language syntax to the code behind it.

Inside the "StandardAdditions.osax" bundle, within /Contents/Resources you'll find the "StandardAdditions.sdef" scripting definition file and the relevant definition line within it:

```xml
<command name="mount volume" code="aevtmvol" description="Mount the specified server volume">
```

Searching in Hopper for "aevtmvol" reveals the function "_AEVTaevtmvol", which has this bit of pseudocode:

```asm
eax = NetFSMountURLSync(var_38, 0x0, 0x0, 0x0, var_3C, esi, var_10);
```

That looks promising!

A Google search turns up documentation for this function via a note from Apple within the [File Manager legacy reference for FSMountServerVolumeSync](https://developer.apple.com/library/mac/documentation/Carbon/Reference/File_Manager/#//apple_ref/c/func/FSMountServerVolumeSync) (_careful loading this page, **tons** of Javascript_), where it says:

_"To mount local volumes and to eject and unmount all volumes, use Disk Arbitration API instead (for more information, see Disk Arbitration Framework Reference). To mount a network volume, use NetFSMountURLAsync instead (to cancel a pending mount request, use NetFSMountURLCancel). **For more information, see NetFS.h in /System/Library/Frameworks/NetFS.framework/Headers**."_

_<small>*(yes, it says NetFSMountURLAsync not NetFSMountURLsync, but just bear with me)</small>_

Ok, cool. Time to go look at some headers.

Within the file mentioned above, we find:

{% gist pudquick/8f09a0b8893c01637b74 01_snip.c %}

That's a bingo.

Looking at the pseudocode, ```0x0``` (null pointer) is used in place of the user and password arguments which lines right up with the documentation _"(overrides URL)"_ indicating that as long as someone provides the password within the URL (example: ```afp://username:password@server/share```), then they're unnecessary.

The only arguments AppleScript passes are _url_, _open_options_, _mount_options_, and a placeholder for getting the returned mountpoint after it succeeds.

So let's look at _open_options_ and _mount_options_ to see what AppleScript put in them.

```asm
00014a1b         lea        eax, dword [ds:eax-0x14621+cfstring_AllowSubMounts] ; @"AllowSubMounts"
```

From the same NetFS.h file above, we can find out what this means:

```C
* kNetFSAllowSubMountsKey = true        Allow a mount from a dir beneath the share point.
#define kNetFSAllowSubMountsKey         CFSTR("AllowSubMounts")
```

Allowing mounting subdirectories sounds like an important setting. Are there any other good ones in there?

```C
* kNAUIOptionKey = UIOption             Suppress authentication dialog UI.
#define kNAUIOptionKey                  CFSTR("UIOption")
// UIOption values                      CFStringRef
#define kNAUIOptionNoUI                 CFSTR("NoUI")
#define kNAUIOptionAllowUI              CFSTR("AllowUI")
#define kNAUIOptionForceUI              CFSTR("ForceUI")
```

Useful!

```C
* kNetFSMountAtMountDirKey = true       Mount on the specified mountpath instead of below it.
#define kNetFSMountAtMountDirKey        CFSTR("MountAtMountDir")
```

Could come in handy!

It's also interesting to note that the AppleScript code provides ```0x0``` for mountpath, which seems to imply that not defining it will make it mount in the standard dynamic fashion at /Volumes.

So at what point do we start turning this into python code?

... How about now? 😆

If you've not dealt with a lot of pyobjc, I feel a little concern as to why you're reading my blog since [that's mostly what I write about](/tag/python). But if you do find yourself in this boat, I can point you to two wonderful writeups about it written by the venerable [Greg Neagle](http://twitter.com/gregneagle/): **[Command-line Tools via Python and Cocoa](https://managingosx.wordpress.com/2015/02/02/command-line-tools-via-python-and-cocoa/)** and **[Accessing More Frameworks with Python](https://managingosx.wordpress.com/2015/02/05/accessing-more-frameworks-with-python-2/)**

The second article I linked there is an important one in relation to this blog post as it's about extending pyobjc on OS X to support working with Frameworks that you can't directly import within python.

Let's see how NetFS handles in this regard:

```python
>>> import NetFS
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named NetFS
```

So let's take a suggestion from that second article as to how to fix this:

{% gist pudquick/8f09a0b8893c01637b74 02_snip.py %}

No errors, very promising! Let's take it for a spin.

We won't define any custom options, we'll just provide a URL to mount and a list (to match the CFArrayRef for our return mountpath).

```python
>>> NetFSMountURLSync(share_url, None, None, None, None, None, [])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: depythonifying 'pointer', got 'list'
```

Ruh roh. It apparently didn't like my list argument.

Swapping it to None doesn't help either.

It's expecting a pointer apparently. How do I get a pointer to a list (or an NSArray) in pyobjc? I know how to do it in ctypes - _but in pyojc???_

**\[INTERMISSION\]**

_At this point I came up with a crazy workaround, which worked, but I'm not putting it into this article, I'll save that for another one. The technique I developed is amazingly useful for other things but is too long to write in the margins here._

**\[END OF INTERMISSION\]**

The real problem boils down to how ```NetFSMountURLSync``` is defined in the bridgesupport file that shipped with OS X, located at: /System/Library/Frameworks/NetFS.framework/Resources/BridgeSupport/NetFS.bridgesupport

{% gist pudquick/8f09a0b8893c01637b74 03_snip.xml %}

That last argument is what's killing us: ```<arg type='^^{__CFArray}'/>```

... a pointer to a pointer to a CFArray ...

... aka a pointer to a CFArrayRef ...

... aka a pointer to a NSArray ([via bridging](https://developer.apple.com/library/ios/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html))

Googling for countless examples on how to use ```NetFSMountURLSync``` show I can pass a null pointer by reference for this argument. OS X isn't expecting me to pass it a CFArrayRef (NSArray), it will <u>build one for me</u> and give me back a pointer to it.

I don't really need to put anything of value _into_ this function argument, I just need to get something _out_ of it.

So without any really good ideas on how to deal with this situation, I emailed the maintainer of the [pyobjc project](http://pythonhosted.org/pyobjc/) - Ronald Oussoren.

He is an _extremely_ kind person and has responded to me the few times I've done this to him before. He definitely came through again this time 😃

His recommendation was that because all of the arguments were basically toll-free bridged types (_pointer to a CFURL_ = _CFURLRef_ = _NSURL_, etc.), it would be best to rewrite the signature as taking a series of object type arguments and then changing the last argument from an input argument into an output.

While pyobjc has problems with double pointer signatures (```^^{__CFArray}``` - each ```^``` indicates a pointer [per the documentation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)), it has no problems with a single level of pointers.

By changing the signature from ```^^{__CFArray}``` (aka _pointer to a CFArrayRef_ aka _pointer to a NSArray_) to ```^@``` (_pointer to an object_), pyobjc becomes able to handle it with ease! Just needed to get from ```^^``` to ```^```.


{% gist pudquick/8f09a0b8893c01637b74 04_snip.py %}

Let's break down that signature: ```i@@@@@@o^@```

The function has 1 return value and 7 input arguments.

[Per the documentation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html), we start off by defining the return value as ```i```, an **int** type, which is pretty common for functions (usually used for error codes).

The arguments follow, starting with 6 ```@``` which indicates 6 **object** type arguments.

Then magic for the last argument: ```o^@```: out-only pointer to an **object** type.

When you provide a signature like this to pyobjc, you still have to pass ```None``` (null pointer) for the argument, but instead of having to deal with dereferencing pointers it saves you the pain _and just moves it to a second return value_.

Here's how you use it:

```python
>>> error, mountpaths = NetFSMountURLSync(share_url, None, None, None, None, None, None)

>>> mountpaths[0]
u'/Volumes/sharename'

```

_Holy cow - IT WORKED AND IT MOUNTED - how freaking cool is that???_

If you'd like a more polished version of the code in this article, including the ability to mount shares at an arbitrary mount path (the directory has to exist in advance) and the use of some of the open and mount options listed above, [you can find it here](https://gist.github.com/pudquick/1362a8908be01e23041d).

It still lacks %-style encoding (like %20 for a space) for username and password portions of the URL. Exercise is left up to the reader to implement 😄

*- mike*
