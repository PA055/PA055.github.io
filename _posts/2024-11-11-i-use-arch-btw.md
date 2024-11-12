---
layout: post
title: I Use Arch BTW
description: My journey in switching from windows to Arch
category:
- Misc
tags:
- arch
- linux
- riceing
date: 2024-11-11 19:02 -0500
---
So I decided to switch to Arch, why you ask? Well for one, windows was getting really slow, it would often freeze up for minutes at a time. The fans on my laptop was also on 24/7 and it was getting really annoying, not to mention disturbing to others in the vicinity. Another reason is that I was using Linux via WSL for almost everything, and it was making much more sense to just switch to Linux. And so I did, I chose Arch in particular because it was a very lightweight OS and i had to do almost everything by myself, at least with regards to setting it up. This let me learn a lot about how Linux actually works internally which was another reason why I switched. 

## Actually Switching
The actually process to install and setup arch was actually pretty straightforward. I just followed the [docs](https://wiki.archlinux.org/title/Installation_guide) and I made it through without too much panic. Well apart from one small little problem. When I was installing I got through disk partitioning and what most people would consider the hard part. But after I shut down and rebooted after that I found that I did not have internet. And I couldn't use `iwctl` to connect to the internet because I hadn't installed it so I kinda panicked a little. But all i had to do was go dig around my basement for a Ethernet cable and plug my computer in, so that was almost disastrous. But after that the rest of the installation was pretty straight forward.  
For formatting my partitions, I decided to go with a [BTRFS](https://en.wikipedia.org/wiki/Btrfs) file system which allowed me to easily create "snapshots" of my file system so when I eventually fuck my system up I can easily revert my system back to before I fucked it up.

## GUI
For my GUI I decided to go with a [Hyprland](https://hyprland.org/) window manager. This is because it offers a lot of customizability when it comes to shortcuts and window management. However it does not come with a UI so I would have to create that myself. For this I decided to go with the [end-4](https://github.com/end-4/dots-hyprland) prebuilt config and customize the parts I don't like myself. I decided to do this because it gave me a good starting point for me to build my rice off of, and especially for my first rice, this was the logical option for me. So I installed it with the script that was provided, and I got a pretty decent starting point, I had a nice set of shortcuts, multiple monitors worked, I had a nice top bar, a decent notification daemon, and a couple of other nice features as well.  

Now to make the GUI how I like it. I couple of things i didn't like about the prebuilt config are that
  1. the top bar is a bit too crowded
  1. the default terminal is fish and the terminal emulator is foot
  1. the default file browser is nautilus
  1. the left side bar with the AI stuff is unnecessary and kinda slows the UI down
  1. the default app picker (there's probably a better name for that) is kinda bad, well at least, I don't like it

So I decided to fix that, and also while I'm at it, add a couple of features that I want to add. But before all of that, I need to setup my login manager so I don't have to login through the shell and manually run hyprland. For this I decided to go with [SDDM](https://github.com/sddm/sddm) as with a nice theme it would look very nice, and it is a highly recommended login manager for hyprland. After I downloaded it, it looked pretty bad, so i had to fix that. I installed the [Astronaut SDDM Theme](https://github.com/Keyitdev/sddm-astronaut-theme) and I changed the image to something I liked better. 

![SDDM Greeter Config](assets/img/SDDM-Greeter.png){: width="600" }

And now I had to make all of my wallpapers match, and so I decided to make a script for that. I needed to change the lock screen wallpaper, the SDDM wallpaper and the normal hyprland wallpaper. The Hyprland wallpaper has support for GIFs but the other 2 don't so if the chosen wallpaper was a GIF i needed to convert it. Other than the fact that I don't know bash, the script wasn't too hard to make, you can see it [here](https://github.com/PA055/dotfiles/blob/main/.config/hypr/scripts/switch_wallpaper) if you so desire. It is very skuffed but i don't change my wallpaper that often so it should be fine. 

### Problem 1: Top Bar

Now to actually start on the changes I planned. Cleaning up the top bar was actually surprisingly easy, all i had to do was delete some files and trace errors until I made them go away, by commenting out random parts of code 🫠. I decided to remove the music info bar because I don't need it, i can just as easily access a better menu by pressing `win`+`m`. So that's one part done, onto the next.

### Problem 2: Terminal Emulator

Next to change my default emulator, for that some googling is all that's requir— or not... Ok so my shortcuts are still opening Foot. Looking though the configs for hyprland, I found that the `win`+`t` keybind maps to opening foot, instead of the system's default terminal emulator, annoying. So i had to search though the configs to replace all of the instances of foot. Ok that's 2 out of 5.

### Problem 3: File Browser

I don't like nautilus, mostly because I like a lot of options, and nautilus looks too clean, like there aren't any options, so I need to change that. My file browser of choice happens to be [thunar](https://docs.xfce.org/xfce/thunar/start) its nice and advanced, learning from the previous task i first changed the keybindings in hyprland, then everything just worked, i didn't have to do anything else, other than setting it up so that it can open a folder in the terminal. 3 out of 5.

### Problem 4: Useless AI stuff

This was pretty easy. I just deleted all of the files that pertained to AI, its gone now! And surprisingly there was only a couple of errors that were easy to fix. 4 out of 5.

### Problem 5: App Launcher

Now to the default app picker, this was the most tightly integrated part, so i kinda just kept it, because i liked some parts of it, but i changed my keybind so pressing `win` opens [walker](https://github.com/abenz1267/walker) my app launcher of choice. It looks clean and has enough features for me to like using it. And that's all of the things i didn't like about the preconfig. Well, all of the major things. I'll come back to this eventually, or just create a new rice ¯\\\_(ツ)\_/¯.

## Neovim

Now onto the other things. First, it's time to setup my code editor. I chose neovim for this rather than going with vscode like I did on windows, this is for a couple of reasons, I've always wanted to learn vim and use it for my editor, and what better way to learn than to throw myself right into it, and it runs much better, with vscode open, my computer's ram spikes through the roof, while neovim barely has an impact. 

One small problem though, neovim has a lot of initial configuration, way too much for me, a beginner ricer. Sooo I decided to use a premade config, [Nvchad](https://nvchad.com/) for the win. Nvchad gave me a good theme, lsp config, and other useful plugins. I did read the documentation to at least know how it was set up. I did add a couple of other plugins though, the list includes Cord.nvim for discord rich presence, todocomments.nvim for better TODO highlighting and searching, and lazygit.nvim because a TUI makes much more sense for git than a CLI. And that was that, a couple of extra mappings to make using neovim better (`ctrl`+`s`) and that's my config.

## Windows

Now I cant just ditch windows, as much as I'd like to. I still need it for a couple of things, most notably, my CAD software, I'm used to Inventor, so I'd like to keep using it. Alas I cannot emulate it using wine, but I do want that to run steam games. So I'm going to need a virtual machine, I decided on using VMware, mostly because I'm using it for CyberPatriot as well. Time to google! So i found a very helpful [Arch wiki article on VMware](https://wiki.archlinux.org/title/VMware) and that pretty much guided me though it, good thing the Arch wiki pages are very detailed. So that was pretty uneventful, now time to make steam games work. So i decided to use bottles because one of my friends recommended it, and it seemed like a good choice, the AUR package pretty much installed everything for me, in like **8 hours**. Yeah... after that wait, it actually worked, I could run steam games (mostly Celeste 🍓).

## Various Applications

Now my rice was almost complete. All I had to do was install various application so I could actually work on my work. A small not comprehensive list: libreoffice suite, thunderbird, pinta, jDownloader, qBittorrent, MATLAB, SquareLine Studio (you'll see what this is for soon), OBS, Unity, GIMP, Discord, and Firefox. And then a bunch more non-gui packages.

So that's my rice, for now, You can find the dotfiles on my github or [here](https://github.com/PA055/dotfiles). I'll probably make a new rice over winter break, or when I get bored of this one, and I'll make the new one without using a preconfig for hyprland. 

_IDK how to end this so here's a ghost 👻_

