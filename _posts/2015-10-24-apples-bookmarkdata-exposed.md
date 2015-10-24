---
summary: "Ever wonder what's inside NSURL BookmarkData? I sure did. Found some secrets!"
layout: post
tags: python osx hopper
date: "2015-10-24 09:00:00"
title: Apple's BookmarkData - exposed!
---

*(And we'll cover `.sfl` files while we're at it!)*

A good friend of mine, [Allister](<https://twitter.com/Sacrilicious>), had an interesting question on the [MacAdmins Slack](<http://macadmins.org/>):

>   *@allister: zero hits on developer.apple.com for this file extension: Library/Application Support/com.apple.sharedfilelist/com.apple.LSSharedFileList.RecentServers.sfl*

To wit: What *are* the `.sfl` files that are appearing in OS X 10.11 El Capitan? There sure are a lot of them!

Diving into the file format, it was pretty obvious it was a binary plist - and with a little more digging, an [NSKeyedArchiver](<https://developer.apple.com/library/watchos/documentation/Cocoa/Reference/Foundation/Classes/NSKeyedArchiver_Class/index.html>) file at that.

Being [very familiar](<http://michaellynn.github.io/2015/07/26/exploring-os-x-preview-signatures/>) with NSKeyedArchiver formats now, it didn't take a whole lot of effort to come up with a script that allowed me to extract the contents of the com.apple.LSSharedFileList.RecentServers.sfl file.

That is - it didn't take a lot of effort to extract the **initial** layer of contents.

{% gist pudquick/c089771fd8196f449662 01_snip.py %}

At its heart, it's a dictionary with 3 keys, the most interesting key being `items`.

The `items` array is made up of `SFLListItem`, a new internal object in 10.11, with the following properties:

-   `name`: Display name in the list (in this case, the name of the volume)

-   `order`: Number representing order within the list

-   `uniqueIdentifier`: A NSUUID generated to uniquely represent the entry in the list

-   `properties`: A dictionary with one or more keys (all of these only had "com.apple.LSSharedFileList.OverrideIcon.OSType" with the value "srvr")

-   `bookmark`: An NSURL BookmarkData object containing information to reach the file/shares

If you look up [Apple's documentation about BookmarkData](<https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/#//apple_ref/doc/uid/20000301-SW34>), you'll find details about how to make them, how to open them, and how to get some *very* basic details from them.

Unfortunately, if you want to just get the share URL that was used to create the BookmarkData in the first place, the only API call Apple offers is [this rather unsavory one](<https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/#//apple_ref/occ/clm/NSURL/URLByResolvingBookmarkData:options:relativeToURL:bookmarkDataIsStale:error:>):

{% gist pudquick/c089771fd8196f449662 02_snip.m %}

I say "unsavory" because of the implementation details:

>   *"<span style="color:red">This method fails</span> if the original file or directory could not be located or is on a volume that could not be mounted.*

>   *If this method fails, you can use the resourceValuesForKeys:fromBookmarkData: method to obtain information about the bookmark, such as the last known path (NSURLPathKey) to help the user decide how to proceed."*

Even with the options `NSURLBookmarkResolutionWithoutMounting` and `NSURLBookmarkResolutionWithoutUI`, you'll find [articles like this one](<http://tangent405.com/dealing-with-slow-security-scoped-bookmarks>) which indicate that the OS is going to attempt to resolve the resource, even if it doesn't pop up anything to the user when it can't find it.

*And yet*, if you open up the BookmarkData in a [hex editor](<http://ridiculousfish.com/hexfiend/>), it's **obvious** the URL information is there in a form it should be directly extractable from. For something like a normal network share, there is absolutely **no reason** to need to do any sort of resolving to be able to return this data. We should be able to just say "Give me the URL" and it should instantly return the value (instead of failing because the server isn't visible on the current network).

So what about the aforementioned `resourceValuesForKeys:fromBookmarkData:` ?

<span style="color:red">USELESS!</span> The NSURLPathKey for server shares appears to return the mounted volume path (`/Volumes/whatever`) - **not the URL of the server share itself**!

… Maybe it's a resource value in a different key?

But suddenly - *more* horrible implementation details: *You can't get the list of available keys!*

Everywhere you search, everyone just recommends ["try all the keys"](<http://www.cocoabuilder.com/archive/cocoa/313438-how-to-get-bookmarks-data-for-non-existing-files.html>) or "try the keys you want values for".

… But [none of the keys](<https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/#//apple_ref/doc/uid/20000301-DontLinkElementID_1>) Apple has documented return just the simple share URL. They apparently want you to only use `URLByResolvingBookmarkData` for that.

To hell with that!
==================

So instead, let's go exploring with [my favorite disassembler](<http://www.hopperapp.com>) 😁

According to Apple's documentation, NSURL is part of Foundation and sure enough we can find the `NSURL.h` header file within `Foundation.framework`. Let's see what Hopper tells us `URLByResolvingBookmarkData` is doing:

{% gist pudquick/c089771fd8196f449662 03_snip.m %}

A quick inspection shows this to be a wrapper around `initByResolvingBookmarkData`, on with the hunt!

{% gist pudquick/c089771fd8196f449662 04_snip.c %}

That CF looks like we're skipping on over to `CoreFoundation.framework`.

{% gist pudquick/c089771fd8196f449662 05_snip.c %}

**Ruh roh!**

This function isn't defined here, it's apparently a "CFCoreServicesInternal" bit of code. Now where could that be defined? …

… Let's check our good friends over at `/System/Library/PrivateFrameworks` 😆

*Oh my*, there appears to be a `CoreServicesInternal.framework` there!

Imagine my surprise.

Let's look inside.

![](</images/2015-10-24-apples-bookmarkdata-exposed/BookmarkData1.png>)

**Wow.** This looks like the place to be!

While we're here, let's take a glance at all the internal things Apple does with BookmarkData …

![](</images/2015-10-24-apples-bookmarkdata-exposed/BookmarkData2.png>)

OH *HELLLLLLOOoo… what do we have here???*

`returnAllPropertiesInBookmark`, `returnAllPropertyKeysInBookmark`, and `returnDetailedDump` you say? *Fascinating* function names.

Visiting any one of those functions, I'm extremely curious to see how they get **called**. So we use Hopper's helpful "Show Places Calling This Procedure…" and all 3 take us to `BookmarkResourcePropertyKeyToInfoStructInit()`

![](</images/2015-10-24-apples-bookmarkdata-exposed/BookmarkData3.png>)

😱 😭 😱

*Do my eyes deceive me?* Are these **secret undocumented NSURL** `resourceValuesForKeys:` keys !??!

They sure as hell look like it! Let's try ‘em out!!

How about we try `NSURLBookmarkAllPropertiesKey`, first?

<small>*(\*insert magical handwaving, skipping over the boring part to get to the amazing part\*)*</small>

{% gist pudquick/c089771fd8196f449662 06_snip.py %}

Good god. It worked. It's *all* there.

All the keys Apple lists - and **a ton that they don't**.

… and yet … none of them are what I'm looking for: just the plain URL to the share.

`NSURLBookmarkAllPropertyKeysKey` turns out to be a bit of a repeat, listing only the keys (but without values).

Want to guess what `NSURLBookmarkDetailedDescription` does?

It <span style="color:red">knocked my socks off</span>, that's what it does.

{% gist pudquick/c089771fd8196f449662 07_snip.txt %}

I've clipped bits out here, but it's a **programmer's debug output** for the entire BookmarkData structure, giving all of the offset values within the data and breaking down *what's actually contained in it*.

Simply amazing. Everything I could have asked for.

The URL of the share itself shows up like this:

`11) itemType=0x2050 flags=0x0 dataOffset=0x148 volMountURL:"<CFURL 0x7fb2b1691d30 [0x7fff767fbed0]>{string = smb://username@192.168.0.1/shares, encoding = 134217984, base = (null)}"`

With the information this undocumented debug output provides, I was able to determine the entire structure of the BookmarkData format, which I'll now document here 😄

The BookmarkData Structure
--------------------------

![](</images/2015-10-24-apples-bookmarkdata-exposed/BookmarkExample.png>)

Here we'll go over this simple 260 byte version of a BookmarkData structure which contains only a single TOC with a single data record. The details I give here were collected after looking at quite a few samples.

*(Note: Unless specified otherwise, all integers are unsigned and little endian)*

### BookmarkData Header

*(all examples appeared to be 48 bytes in length)*

![](</images/2015-10-24-apples-bookmarkdata-exposed/Structure_01_Header.png>)

<span style="color:red">■</span> 4 byte string: **BookmarkData signature** (can be `book`, the new style, or `alis`, the original alias record of Mac OS)

<span style="color:lightgreen">■</span> 32-bit integer: **Total length of BookmarkData** structure (including header)

<span style="color:blue">■</span> 4 bytes: **Version** *(might be a big endian int*? `1040`*)*

<span style="color:yellow">■</span> 32-bit integer: **Offset of BookmarkData data** payload from beginning of BookmarkData structure (always `48` currently)

<span style="color:cyan">■</span> N-bytes: null / `0x00` bytes used as **filler** until the beginning of BookmarkData data payload

### BookmarkData Data

The data portion is a combination of several things: TOC (table of contents) records, data records (of various kinds), and most importantly the pointer to the first TOC. The "Offset of BookmarkData data" value points to the beginning of this data structure.

![](</images/2015-10-24-apples-bookmarkdata-exposed/Structure_02_Data.png>)

<span style="color:orange">■</span> 32-bit integer: **Offset of first TOC**, measured from the start of the BookmarkData data payload (example: an offset of `100` is actually `148` bytes from the beginning. `48` byte header + `100` bytes from start of data payload)

<span style="color:lightgray">■</span> N-bytes: The remainder of the data section is composed of TOC records (which tell where to find the data records) and the data records themselves.

### BookmarkData TOC

BookmarkData has the concept of a "first TOC", the one pointed to by the beginning of the BookmarkData data payload. Each TOC has information about a number of data records as well as information on the offset of the "next TOC". If there is no next TOC, the offset for "next TOC" will be 0. Each TOC is comprised of a header and a data section.

![](</images/2015-10-24-apples-bookmarkdata-exposed/Structure_03_TOC.png>)

### TOC Header

<span style="color:red">■</span> 32-bit integer: **Length of TOC data** segment after the header

<span style="color:lightgreen">■</span> 16-bit integer: **Record type (TOC)** (`0xFEFF` / `65279`. Might be signed? In which case the value would be `-2`)

<span style="color:blue">■</span> 16-bit integer: **Flags** (unused for TOC, always set to `0xFFFF`)

### TOC Data

<span style="color:yellow">■</span> 32-bit integer: **Level** (for a server mount, the value seems to be `1`)

<span style="color:cyan">■</span> 32-bit integer: **Offset of next TOC** record, measured from beginning of BookmarkData data section

<span style="color:magenta">■</span> 32-bit integer: **Number of records** in this TOC

*[BEGIN: N number of TOC data records]*

### TOC Data Record

<span style="color:orange">■</span> 16-bit integer: **Record type** - varies (examples: `8272`: "volMountURL", `8208`: "volName")

<span style="color:salmon">■</span> 16-bit integer: **Flags** (always seems to be `0x0000`)

<span style="color:lightblue">■</span> 64-bit integer: **Offset of record data**, measured from beginning of BookmarkData data portion

*(Note: That may actually be a 32-bit integer, followed by 4 null bytes - I would be surprised to see 64-bit offsets here when TOC offsets are 32-bit)*

*[END TOC data records]*

### Standard Data Record

These are the records pointed to by TOCs. They represent the bulk of the contents of BookmarkData and are where the real data is stored.

![](</images/2015-10-24-apples-bookmarkdata-exposed/Structure_04_Records.png>)

<span style="color:red">■</span> 32-bit integer: **Length of data** payload, the actual data stored here

<span style="color:blue">■</span> 32-bit integer: **Data type** - varies (examples: `2305`: CFURL, `257`: string)

<span style="color:lightgreen">■</span> N-bytes: **Record data**, padded with `0x00` on the end if necessary to reach a multiple of 4 bytes in length

(… wow, you made it this far? Yay! 🎉)

Some of you readers with a keen eye may have recognized the data coloring abilities of [Synalyze It!](<http://www.synalysis.net>) and maybe you're secretly hoping I've written up a formal grammar for BookmarkData using it … sadly I have not 😞. I'm brand new to the software and unfortunately the interface is not very intuitive when it comes to interactively creating a new grammar. I haven't decided yet whether I'm to blame or poor UI is.

… The *even sharper* readers which have persisted this far may have noticed that the standard data record in my example doesn't appear to be something you can easily extract a share URL from. There's a reason for that!

I picked the older `alis` BookmarkData structure as an example because of its size (260 bytes for this one). Your average `book` modern style BookmarkData structure is well over a megabyte in size (!) because one of the records it contains is `icns` icon information for how your share's icon should look.

Unfortunately, the `alis`-style is literally the thinnest of wrappers around what's actually an original Mac OS (pre OS X) `alis` resource fork fileshare record.

![](</images/2015-10-24-apples-bookmarkdata-exposed/ResEdit.gif>)

The newer `book` variant contains the record types (like `8272` aka "volMountURL") I mentioned earlier.

So - what to do?

Well. 
Back to Where We Started
------------------------


If you have records that are `book` type, [I have posted quick and dirty code here](<https://gist.github.com/pudquick/4776b4b2075bf9b7e512>) that will give you the "volMountURL" information instantly for the `.sfl` files that started this wild ride.

I'm working on more formal code for BookmarkData parsing as I write this. But now that the information is out there, maybe someone will beat me to the punch 😊

If you have `alis`-style information, you can can attempt to use code like this:

`url,isStale,err = NSURL.URLByResolvingBookmarkData_options_relativeToURL_bookmarkDataIsStale_error_(bookmark_data, NSURLBookmarkResolutionWithoutMounting+NSURLBookmarkResolutionWithoutUI, None, None, None)`

… But don't be surprised if you get back results like this if the resource no longer exists:

{% gist pudquick/c089771fd8196f449662 08_snip.py %}

But worry not, friend readers - I've *also* completely decoded `alis` records in python, [as have others](https://bitbucket.org/al45tair/mac_alias)!

And the AFPX mount records they can contain.

And even resource forks!

We'll get to it in another blog post 😆

*- mike*
