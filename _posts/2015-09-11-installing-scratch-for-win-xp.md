---
comments: true
comments: true
comments: true
layout: post
title:  "Scratch with Johnny-five"
tags: Visual Programming
---
comments: true
comments: true
comments: true

# Installing Scratch for Windows XP

## WHY?

Well because Scratch is used by teachers and students. They are usually found in places called schools. Schools may not have funding to buy latest models of PC frequently so some of them are still running XP. That's why.

# What do you need?

- [Scratch version 439.1](https://scratch.mit.edu/scratchr2/static/sa/Scratch-439.1.exe)

Scratch 2 Offline [documentation](https://scratch.mit.edu/scratch2download/) suggests if *Scratch 2 crashes* on startup, you should reinstall. It does not say anything about *computer crash*. FYI I did try reinstall and it did not help. The crash is consistent.

# What Else do you need?

DO ***NOT*** update

DO ***NOT*** update

DO ***NOT*** update

Important things are worth repeating thrice.

But Adobe Air nudges you every now and then, and Scratch does this everytime on startup.

It is hard to imagine, in a school environment, nobody would be attempted to click on OK and see what happens.

We spent too much time on this to know, leave no such things in the hands of the users.

## Disable Adobe Air Auto Update

They do have a way to disable automatic update. It requires either a file added to the user's application path, or registry editing. [See link](https://forums.adobe.com/docs/DOC-1321)

Also, disable update ***Immediately*** after Air installation. Once it gets the taste of the later versions, it is not even possible to install Scratch without updating Air, which would again crash the computer.


## Disable Scratch 2 Auto Update

Scratch checks for the update version from Scratch's CDN site. Easy way is to modify hosts file and lead the CDN to localhost. But that would make the official site really ugly because it also hosts all the js and css files of the site.

Second workaround is tricking Scratch to think it actually has a higher version.
Edit the `%ProgramFiles(x86)%\Scratch 2\META-INF\AIR\application.xml` and change the `<VersionNumber>` content to 999. This is as high as it can get. Don't try 9999 or 99999. RTFM.

## Two Stage Installation

Actually, "two stage" is euphemism. More like "intentionally fail on the first run then reinstall".

1. Install Scratch with default options. It will fire up Scratch after installation and on Windows XP, it will probably cause the blue screen of death.
2. Manually remove the `C:\Program Files\Scratch 2` folder. Then install Scratch 2 again, but this time, uncheck the `Run application after installation` checkbox.

Then you should be able to get Scratch running. Under certain conditions.

## Conditions You Say?

Yes. Scratch somehow still dies if you lauch it from the desktop shortcut. Instead, grab a Scratch 2 project file( with `.sb2` file extension) and double click that file to open Scratch.

## Disclaimer

Above procedures are tested on XP Professional Simplified Chinese version. Other versions like Home and Enterprise have not been tested. Neither have other language packs.



