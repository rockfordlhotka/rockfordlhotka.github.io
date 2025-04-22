---
layout: post
title: Running Linux on My Surface Go
date: 2025-04-22T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
![Surface Go running Ubuntu]({{ site.url }}/assets/2025-04-22-Running-Linux-on-My-Surface-Go/PXL_20250422_195349182.jpg)

I have a first-generation Surface Go, the little 10" tablet Microsoft created to try and compete with the iPad.

I'll confess that I never used it a lot. I _tried_, I really did! But it is underpowered, and I found that my Surface Pro devices were better for nearly everything.

My reasoning for having a smaller tablet was that I travel quite a lot, more back then than now, and I thought having a tablet might be nicer for watching movies and that sort of thing, especially on the plane.

It turns out that the Surface Pro does that too, without having to carry a second device. Even when I switched to my Surface Studio Laptop, I _still_ didn't see the need to carry a second device - though the Surface Pro is absolutely better for traveling in my view.

I've been saying for quite some time that I think people need to [look at Linux as a way to avoid the e-waste involved in discarding their Windows 10 PCs](https://blog.lhotka.net/2024/12/15/Do-not-throw-away-your-old-PCs) - the ones that can't run Windows 11. I use Linux regularly, though usually via the command line for software development, and so I thought I'd put it on my Surface Go to gain real-world experience.

> I have quite a few friends and family who have Windows 10 devices that are perfectly good. Some of those folks don't want to buy a new PC, due to financial constraints, or just because they don't think their current PC is a problem. End of support for Windows 10 is a problem!

The Surface Go is a bit trickier than most mainstream Windows 10 laptops or desktops, because it is a convertable tablet with a touch screen and specialized (rare) hardware - as compared to most of the devices in the market. So I did some reading, and used Copilot, and found a decent (if old) [article on installing Linux on a Surface Go](https://www.zdnet.com/article/how-i-put-linux-on-a-microsoft-surface-go-in-just-an-hour/).

> ⚠️ One quick warning: Surface Go was designed around Windows, and while it does work reasonably well with Linux, it isn't as good. Scrolling is a bit laggy, and the cameras don't have the same quality (by far). If you want to use the Surface Go as a small, lightweight laptop I think it is pretty good; if you are looking for a good _tablet_ experience you should probably just buy a new device.

Fortunately, Linux hasn't evolved all that much or all that rapidly, and so this article remains pretty valid even today.

## Using Ubuntu Desktop

I chose to install Ubuntu, identified in the article as a Linux distro (distribution, or variant, or version) that has decent support for the Surface Go. I also chose Ubuntu because this is normally what I use for my other purposes, and so I'm familiar with it in general.

However, I installed the latest [Ubuntu Desktop (version 25.04)](https://ubuntu.com/download/desktop), not the older version mentioned in the article. This was a good choice, because support for the Surface hardware has improved over time - though the other steps in the article remain valid.

## Download and Set Up Media

The steps to get ready are:

1. [Download Ubuntu Desktop](https://ubuntu.com/download/desktop#how-to-install-NobleNumbat) - this downloads a file with a `.iso` extension
2. Download software to create a bootable flash drive based on the `.iso` file. I used software called [Rufus](https://rufus.ie/en/) - just be careful to avoid the flashy (spammy) download buttons, and find the actual download link text in the page
3. Get a flash drive (either new, or one you can erase) and insert it into your PC
4. Run rufus, and identify the `.iso` file and your flash drive
3. Rufus will write the data to the flash drive, and make the flash drive bootable so you can use it to install Linux on any PC
6. 🛑 BACK UP ANY DATA on your Surface Go; in my case all my data is already backed up in OneDrive (and other places) and so I had nothing to do - but this process WILL BLANK YOUR HARD DRIVE! 🛑

## Install Ubuntu on the Surface

At this point you have a bootable flash drive and a Surface Go device, and you can do the installation. This is where the zdnet article is a bit dated - the process is smoother and simpler than it was back then, so just do the install like this:

1. 🛑 BACK UP ANY DATA on your Surface Go; in my case all my data is already backed up in OneDrive (and other places) and so I had nothing to do - but this process WILL BLANK YOUR HARD DRIVE! 🛑
2. Insert the flash drive into the Surface USB port (for the Surface Go I had to use an adapter from type C to type A)
3. Press the Windows key and type "reset" and choose the settings option to reset your PC
4. That will bring up the settings page where you can choose Advanced and reset the PC for booting from a USB device
5. What I found is that the first time I did this, my Linux boot device didn't appear, so I rebooted to Windows and did step 4 again
6. The second time, an option was there for Linux. It had an odd name: Linpus (as described in the zdnet article)
7. Boot from "Linpus" and your PC will sit and spin for quite some time (the Surface Go is quite old and slow by modern standards), and eventually will come up with Ubuntu
8. The thing is, it is _running_ Ubuntu, but it hasn't _installed_ Ubuntu. So go through the wizard and answer the questions - especially the wifi setup
9. Once you are on the Ubuntu (really Gnome) desktop, you'll see an icon for _installing_ Ubuntu. Double-click that and the actual installation process will begin
10. I chose to have the installer totally reformat my hard drive, and I recommend doing that, because the Surface Go doesn't have a huge drive to start with, and I want all of it available for my new operating system
11. Follow the rest of the installer steps and let the PC reboot
12. Once it has rebooted, you can remove the flash drive

## Installing Updates

At this point you should be sitting at your new desktop. The first thing Linux will want to do is install updates, and you should let it do so.

I laugh a bit, because people make fun of Windows updates, and Patch Tuesday. Yet all modern and secure operating systems need regular updates to remain functional and secure, and Linux is no exception.

Whether automated or not, you should do regular (at least monthly) updates to keep Linux secure and happy.

## Installing Missing Features

Immediately upon installation, Ubuntu 25.04 seems to have very good support for the Surface Go, including multi-touch on the screen and trackpad, use of the Surface Pen, speakers, and the external (physical) keyboard.

What doesn't work right away, at least what I found, are the cameras or any sort of onscreen/soft keyboard. You need to take extra steps for these.

The zdnet article is helpful here.

### Getting the Cameras Working

The zdnet article walks through the process to get the cameras working. I actually think the camera drivers are now just part of Ubuntu, but I did have to take steps to get them working, and even then they don't have great quality - this is clearly an area where moving to Linux is a step backward.

At times I found the process a bit confusing, but just plowed ahead figuring I could always reinstall Linux again if necessary. It did work fine in the end, no reinstall needed.

1. Install the Linux Surface kernel - which sounds intimidating, but is really just [following some steps as documented in their GitHub repo](https://github.com/linux-surface/linux-surface/wiki/Installation-and-Setup#debian--ubuntu); other stuff in the document is quite intimidating, but isn't really relevant if all you want to do is get things running
2. That GitHub repo also has information about the various camera drivers for different Surface devices, and I found that to be a bit overwhelming; fortunately, it really amounts to just [running one command](https://github.com/linux-surface/linux-surface/wiki/Camera-Support#ubuntu)
3. Make sure you also [run these commands](https://github.com/linux-surface/linux-surface/wiki/Camera-Support#ensure-your-user-account-has-permissions) to give your Linux account permissions to use the camera
4. At this point I was able to follow [instructions to run `cam`](https://github.com/linux-surface/linux-surface/wiki/Camera-Support#test-with-cam) and see the cameras - including some other odd entries I igored
5. And I was able to run `qcam`, which is a command that brings up a graphical app so you can see through each camera

> ⚠️ Although the cameras technically work, I am finding that a lot of apps still don't see the cameras, and in all cases the camera quality is quite poor.

### Getting a Soft or Onscreen Keyboard

Because the Surface Go is _technically_ a tablet, I expected there to be a soft or onscreen keyboard. It turns out that there is a primitive one built into Ubuntu, but it really doesn't work very well. It is pretty, but I was unable to figure out how to get it to appear via touch, which kind of defeats the purpose (I needed my physical keyboard to get the virtual one to appear).

I found an article that has some good suggestions for [Linux onscreen keyboard (OSK) improvements](https://ubuntuhandbook.org/index.php/2022/05/enable-on-screen-keyboard-ubuntu-22-04/). I used what the article calls "Method 2" to install an Extension Manager, which allowed me to install extensions for the keyboard.

1. Install the Extension Manager `sudo apt install gnome-shell-extension-manager`
2. Open the Extension Manager app
3. This is where the article fell down, because the extension they suggested doesn't seem to exist any longer, and there are numerous other options to explore
4. I installed an extension called "Touch X" which has the ability to add an icon to the upper-right corner of the screen by which you can open the virtual keyboard at any time (it can also do a cool ripple animation when you touch the screen if you'd like)
5. I also installed "GJS OSK", which is a replacement soft keyboard that has a lot more configurability than the built-in default; you can try both and see which you prefer

## Installing Important Apps

This section is mostly editorial, because I use certain apps on a regular basis, and you might use other apps. Still, you should be aware that there are a couple ways to install apps on Ubuntu: snap and apt.

The "snap" concept is specific to Ubuntu, and can be quite nice, as it installs each app into a sort of sandbox that is managed by Ubuntu. The "app store" in Ubuntu lists and installs apps via snap.

The "apt" concept actually comes from Ubuntu's parent, Debian. Since Debian and Ubuntu make up a very large percentage of the Linux install base, the `apt` command is extremely common. This is something you do from a terminal command line.

Using snap is very convenient, and when it works I love it. Sometimes I find that apps installed via snap don't have access to things like speakers, cameras, or other things. I think that's because they run in a sandbox. I'm pretty sure there are ways to address these issues - my normal way of addressing them is to uninstall the snap and use `apt`.

### My "Important" Apps

I installed apps via snap, apt, and as PWAs.

#### Snap and Apt Apps

Here are the apps I installed right away:

1. [Microsoft Edge browser](http://microsoft.com/en-us/edge) - because I use Edge on my Windows devices and Android phone, I want to use the same browser here to sync all my history, settings, etc. - I installed this using the default Firefix browser, then switched the default to Edge
2. [Visual Studio Code](https://code.visualstudio.com) - I'm a developer, and find it hard to imagine having a device without some way to write code - and I use vscode on Windows, so I'm used to it, and it works the same on Linux - I installed this as a snap via App Center
3. [git](https://git-scm.com/downloads/linux) - again, I'm a developer and all my stuff is on GitHub, which means using git as a primary tool - I installed this using `apt`
4. [Discord](https://discord.com/) - I use discord for many reasons - talking to friends, gaming, hosting the [CSLA .NET](https://cslanet.com) Discord server - so it is something I use all the time  - I installed this as a snap via App Center
5. [Thunderbird Email](https://www.thunderbird.net) - I'm not sold on this yet - it seems to be the "default" email app for Linux, but feels like Outlook from 10-15 years ago, and I do hope to find something a lot more modern  - I installed this as a snap via App Center
6. [Copilot](https://copilot.microsoft.com/) Desktop - I've been increasingly using Copilot on Windows 11, and was delighted to find that Ken VanDine wrote a Copilot shell for Linux; it is in the App Center and installs as a snap, providing the same basic experience as Copilot on Windows or Android - I installed this as a snap via App Center
7. [.NET SDK](https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?tabs=dotnet9&pivots=os-linux-ubuntu-2410) - I mostly develop using .NET and Blazor, and so installing the .NET software developer kit seemed obvious; Ubuntu has a snap to install version 8, but I used apt to install version 9

#### PWA Apps

Once I got Edge installed, I used it to install a number of progressive web apps (PWAs) that I use on nearly every device. A PWA is an app that is installed and updates via your browser, and is a great way to get cross-platform apps.

Exactly how you install a PWA will vary from browser to browser. Some have a little icon when you are on the web page, others have an "install app" option or "install on desktop" or similar.

The end result is that you get what appears to be an app icon on your phone, PC, whatever - and when you click the icon the PWA app runs in a window like any other app.

1. [Elk](https://elk.zone) - I use Mastodon (social media) a lot, and my preferred client is Elk - fast, clean, works great
2. [Bluesky](https://bsky.app/) - I use Bluesky (social media) a lot, and Bluesky can be installed as a PWA
3. [LinkedIn](https://linkedin.com) - I use LinkedIn quite a bit, and it can be installed as a PWA
3. [Facebook](https://facebook.com) - I still use Facebook a little, and it can be installed as a PWA

#### Using Microsoft 365 Office

Most people want the edit documents and maybe spreadsheets on their PC. A lot of people, including me, use Word and Excel for this purpose. Those apps aren't available on Linux - at least not directly. Fortunately there are good alternatives, including:

1. Use https://onedrive.com to create and edit documents and spreadsheets in the browser
2. Use https://office.com to access Office online if you have a subscription
3. Install [LibreOffice](https://www.libreoffice.org/), an open-source office productivity suite sort of like Office

I use OneDrive for a lot of personal documents, photos, etc. And I use actual Office for work. The LibreOffice idea is something I might explore at some point, but the online versions of the Office apps are usually enough for casual work - which is all I'm going to do on the little Surface Go device anyway.

One feature of Edge is the ability to have multiple profiles. I use this all the time on Windows, having a personal and two work profiles. This feature works on Linux as well, though I found it had some glitches.

My default Edge profile is my personal one, so all those PWAs I installed are connected to that profile.

I set up another Edge profile for my CSLA work, and it is connected to my marimer.llc email address. This is where I log into the M365 office.com apps, and I have that page installed as a PWA. When I run "Office" it opens in my work profile and I have access to all my work documents.

On my personal profile I don't use the Office apps as much, but when I do open something from my personal OneDrive, it opens in that profile.

The limitation is that I can only edit documents while online, but for my purposes with this device, that's fine. I can edit my documents and spreadsheets as necessary.

## Conclusion

At this point I'm pretty happy. I don't expect to use this little device to do any major software development, but it actually does run vscode and .NET just fine (and also Jetbrains Rider if you like that).

I mostly use it for browsing the web, discord, Mastodon, and Bluesky.

Will I bring this with when I travel? No, because my normal Windows 11 PC does everything I want.

Could I live with this as my "one device"? Well, no, but that's because it is underpowered and physically too small. 

But could I live with a modern laptop running Ubuntu? Yes, I certainly could. I wouldn't _prefer_ it, because I like full-blown Visual Studio and way too many high end Steam games.

The thing is, I am finding myself leaving the Surface Go in the living room, and reaching for it to scan the socials while watching TV. Something I could have done just as well with Windows, and can now do with Linux.