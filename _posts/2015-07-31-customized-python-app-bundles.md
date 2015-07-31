---
summary: "Changing how python looks in OS X - buh-bye rocketship!"
layout: post
tags: python osx
date: "2015-07-31 11:45:00"
title: Customized Python.app Bundles 
---

The python that ships with OS X is not managed by [python.org](http://python.org).

It is a custom Apple-maintained distribution of python - much in the same way that Apple used to have their own distribution of Java included in OS X (Apple Java is still available, but has been deprecated and doesn't ship with OS X - [10.11 will be the last version to ever have it](https://developer.apple.com/library/prerelease/mac/releasenotes/General/rn-osx-10.11/)).

As such, Apple's python has some customizations:

* ```/usr/bin/python``` is not actually the real "python" binary. It is actually a stub binary that calls the real python interpreter that your OS is configured to use (for more configuration details, see [my AFP548 article](https://www.afp548.com/2013/04/25/python-versioning-in-os-x/))
* It ships with many third-party modules pre-installed that are not part of the core python distribution, including the lovely [pyobjc](https://pythonhosted.org/pyobjc/), a personal favorite of mine.
* ... and because they want to encourage python \<-\> Objective-C bridging, there is a "Python.app" bundle that's included, allowing OS X APIs that need application bundle details (Dock icon, display name, etc.) to function. This is where we get the lovely "rocketship", if you've ever seen it before:

![](</images/2015-07-31-customized-python-app-bundles/rocketship.png>)

Unfortunately, the "Python.app" bundle is located in a subdirectory within /System/Library/Frameworks/Python.framework.

This means you *really shouldn't* mess with it or change it - making customization of how python presents itself when it appears in the Dock as a GUI application really annoying. And in 10.11, with the advent of [System Integrity Protection](https://developer.apple.com/library/prerelease/mac/releasenotes/MacOSX/WhatsNewInOSX/Articles/MacOSX10_11.html#//apple_ref/doc/uid/TP40016227-DontLinkElementID_19) it will be **extremely painful** to make modifications, even if you wanted to.

But SIP isn't even the biggest issue facing python in 10.11, it's just made the real culprit worse.

The big baddie coming up is: [App Transport Security](https://developer.apple.com/library/prerelease/mac/releasenotes/MacOSX/WhatsNewInOSX/Articles/MacOSX10_11.html#//apple_ref/doc/uid/TP40016227-DontLinkElementID_18)

Why is ATS a problem you might ask?

Well, if you write any python code currently on OS X that relies on the NSURL APIs via pyobjc - *without performing some major voodoo*, you're now going to be restricted to **only** HTTPS connections (and only those that meet the strict/higher encryption requirements of ATS)!

The reason for this is that disabling ATS to allow the use of HTTP URLs via NSURL APIs requires [special keys in the Info.plist of the application bundle](https://developer.apple.com/library/prerelease/ios/technotes/App-Transport-Security-Technote/) ... **which are not present in the Apple-provided Python.app**.

And since you really can't make changes to the embedded Info.plist (and any changes would go away, potentially, with an OS update), this gets pretty painful pretty quickly.

Several of us that work on [munki](https://github.com/munki/munki) learned about this situation when Developer Preview 5 of 10.11 was released (when ATS became enabled), as munki made the switch to using NSURL APIs quite some time ago.

Out of that research, we did figure out [a crazy fix](https://github.com/munki/munki/commit/1dd8329d665d1d724ddc56ea703552effcd42db8) that makes munki work again - but some alternative approaches were also discovered, one of which was suggested by [Pepijn Bruienne](https://twitter.com/bruienne/):

* It is possible to "stub out" an alternate Python.app bundle which symlinks the /System/Library/Framework/Python.framework resources - **except for** Info.plist.

You can provide your own Info.plist and then execute python from within this alternate application bundle and OS X will use your customizations instead! Not only could you override ATS if you wanted to, now you can customize the icon in the Dock, the name in the menubar, the bundle identifier of your python instance - the possibilities are endless.

So, to that effect, I wrote a piece of code that quickly stubs out a new Python.app bundle to a location of your choice, complete with icon customization and Info.plist overrides:

{% gist pudquick/8e8e09b8c1f5ceb5020b 01_snip.py %}

To use it as it's written here, you'd create a TempApp instance with your customizations and then call it like a normal external executable with [subprocess](https://docs.python.org/2/library/subprocess.html) and the appropriate arguments (like some code.py file, etc.):

{% gist pudquick/8e8e09b8c1f5ceb5020b 02_snip.py %}

... but this is me writing this, so you *know* I didn't stop there.

Python includes a another fantastic module called multiprocessing and I especially love the workflow of the [Process class](https://docs.python.org/2/library/multiprocessing.html#the-process-class) in it.

It allows you to spin up another python interpreter trivially in a new process and easily share data between this child process and the original with things like [Queues and Pipes](https://docs.python.org/2/library/multiprocessing.html#exchanging-objects-between-processes). Additionally, you don't need a chunk of python code in a file - you can just pass a function to the Process class as the ```target``` argument and it will use that for the core of execution.

It's a lovely lovely module and I'd recommend using it for any time you'd need to spin up additional python instances ...

... except in the specific case of our situation: running a python interpreter from a different path

The problem is: For python 2.6+ for Windows - [they make it trivial to use python at a different path](https://docs.python.org/2/library/multiprocessing.html#multiprocessing.set_executable).

For OS X and other Unix platforms, they didn't add that support **[until python 3.4](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.set_executable)**.

**GAHHHH!** So annoying.

... So, I updated my code and added some capability to do that ...

In order to do this, I used the capability of the [marshal](https://docs.python.org/2/library/marshal.html) module to turn python code into a semi-portable (within same python builds on same architecture - not a problem in our situation) data stream that can be passed to another python instance and turned back into functional code.

I also used the [pickle](https://docs.python.org/2/library/pickle.html) module to do the same for sending and receiving data between the processes.

This allowed me to pass my python code to the child process over stdin (which keeps the code out of the argument list, keeping the output of tools like ```ps``` and ```top``` uncluttered).

{% gist pudquick/bd98a589dbe0ddf17cd4 %}

It's crazy code. But the results are pretty awesome.

![](</images/2015-07-31-customized-python-app-bundles/running.png>)

![](</images/2015-07-31-customized-python-app-bundles/temp_loc.png>)

As you can see, I have full control over the visual representation of python - in the Dock, menubar, and even in application switching and Force Quit dialogs.

Hope you found this as interesting as I did!

*- mike*
