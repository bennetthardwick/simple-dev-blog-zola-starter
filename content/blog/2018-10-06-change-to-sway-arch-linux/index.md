+++
title = "Changing from X.org and i3wm to Sway and Wayland"
description = "Today I decided to embrace the dark side and move from i3wm to Sway... Why didn't I do this sooner!"
aliases = ["/blog/2018-10-06-change-to-sway-arch-linux/"]
[taxonomies]
tags = [ "archlinux" ]
+++

I decided to move my Arch Linux i3wm configuration over to Sway today. Partly because I was bored, but mainly because everything has been feeling really sluggish lately.

I started to notice problems when using VS Code. I couldn't really describe what was going on, but when I typed it just didn't quite feel like I was typing in real time. I wasn't really suprised that things were feeling quite slow, I have quite a unique graphics setup - R9 390x + GTX 770 (GPGPU) - and it's not the first time my franken-PC has played up.

As I was only noticing issues when I was developing, the first iteration of my solution was to just to exclusively use `nvim`.

I've been using the VS Code vim bindings for a quite a while, so moving entirely to Neovim wasn't really an obstacle for me. The only issue that I do a lot of Angular development, and the amazing [Angular Language Service](https://github.com/angular/vscode-ng-language-service) plugin that's available for VS Code doesn't exist for nvim (but I'm working on [changing that](https://github.com/bennetthardwick/nvim-ng-language-service), slowly). Also, Typescript completion in Neovim is quite slow, but that's not a massive issue.

So I used Neovim for a few weeks, and everything was going well. However! As the speed / performance of my editor was amazing, it allowed me to realise how laggy my window manager - i3wm - was.

This really confused me! The whole reason I used (apart from the tiling window manager aspect), was that it was supposed to be incredibly minimalistic and bare-bones. In fact, `i3-gaps`, the the flavour I was using, is only 1.3MiB!

## Enter Wayland

It was around this time that I started investigating Wayland. I still don't know much about it, but everyone seems to say it's the future, and will eventually replace X. A quick skim of [the Wayland architecture page](https://wayland.freedesktop.org/architecture.html) tells me that the main difference is the emphasis on compositing. Unlike X, the compositor is built right into the display server / is the display server.

I'd also heard a little bit about [Sway](https://github.com/swaywm/sway), a tiling window manager / compositor for the Wayland display protocol. It looked pretty neat, and was /apparently/ 100% compatible with i3wm. Why not try it out?

Now, I was a little bit hesitant to change. Apart from not being to say "hey I use i3wm btw" anymore, there would obviously be quite a lot of reconfiguration that I would have to perform. Regardless, I took the plunge.

## Install Sway, now reboot.

The first step of installing sway was, well, installing sway. In Arch Linux, this is [quite easily done](https://wiki.archlinux.org/index.php/Sway), by installing some packages from the [AUR](https://wiki.archlinux.org/index.php/Arch_User_Repository). They're the [latest version of sway](https://aur.archlinux.org/packages/sway-latest-git/), and the [wlroots Wayland compositor library](https://aur.archlinux.org/packages/wlroots-git/).

After installing those packages, I'd also suggest removing your display manager (if you're using one). According to the wiki, Sway doesn't have fantastic support for display managers, and the majority of people that use it just start it from `tty1`, I decided to do the same.

For me, it was just a matter of removing the `lightdm` and `lightdm-gtk-greeter` packages.

Great! Now go a head and reboot your computer.

### Error! Invalid i3 config!

Well well well! Apparently it doesn't "work out of the box" like you suggested, Sway.

Oh well! There were only three lines that it complained about. For me, it was just a case of editing my config in three ways:

1. Change `new_window 1pixel` to `default_border pixel 1`

2. It seems that Sway is a lot stricter than i3 when it comes to rules. In my config it was throwing an error at `for_window[class=".*"]`. All I had to do was change it to `for_window [class=".*"]`.

3. It was getting mad about a few styles that I had set. Simply deleting `client.background` and `client.placeholder` fixed all my issues.

### Polybar to Waybar

Polybar simply will not work in Sway, at least from what I've tried. Luckily, there's a fantastic program (event better than Polybar imho) called Waybar that you can use.

Configuring it could not be easier! It takes a basic JSON file to set fonts, modules and what not, and also a CSS style sheet that handles all the stuff that CSS does.

You can have a look at my [dotfiles](https://github.com/bennetthardwick/dotfiles/tree/master/.config/waybar) if you want more about how it works.

### Saying Goodbye to Unicode RXVT

This is really sad. I've been using URxvt ever since I started using Arch Linux, and it would be a lie to say I haven't grown fond of it. Unfortunately, there were a few noticeable issues for me, mainly flashing / stuck characters. There was also an error with fonts not loading - it just had to go.

So on a whim, I decided to try using [termite](https://github.com/thestinger/termite), and what would you know. It worked great!

Moving my Gruvbox and Fantasque Sans Mono font theme across required me to create a termite configuration file. Unfortunately there were only 16 color themes in the [Gruvbox contrib](https://github.com/morhetz/gruvbox-contrib/tree/master/termite) repo, so I used a Vim macro to quickly change the syntax of my `.Xresources` file to suit termite.

One of the issues I had with URxvt was that the Gruvbox theme for Neovim was a few shades off, unless I used a [special build](https://aur.archlinux.org/packages/rxvt-unicode-256xresources). Luckily there was no issue with termite.

## Impressions

I was super impressed when I first started using Sway! Everything was so smooth, there are no artifacts when I open and close windows, VS Code feels really snappy, and everything just looks nicer.

It isn't without problems though.

Now that VS Code is fine, Chromium has started to feel really slow. I'm starting to think that my computer is just messing with me! Changing to Firefox seems to work, but I think there's an underlying problem at hand.

Since Sway does all the work, I have to rely on the graphical optimisations and configuration that it provides me. I haven't had much time to explore just yet, but I feel it might take a bit of work to make everything feel seamless.

But until then, I'll definitely keep using Sway! It's an obvious improvement over i3wm and a fantastic step towards wider adoption of Wayland.

<br />

---

## The Next Day

I've been using Sway for the first part of today and everything has been pretty good, but there are some things that are a bit annoying.

Firstly, XWayland is pretty trash. All the apps that need the X compatibility layer to run (firefox, chromium, etc) aren't fantastic. I've started trying to build the [firefox-wayland](https://aur.archlinux.org/packages/firefox-wayland/) package from the AUR, so I'll see how that goes.

On the other hand, GTK+ applications are working great! I tried swapping them over to use the X11 backend and the difference is astounding. +10 for Wayland.

One issue I did run in to was that my GTK theme wasn't being used. After a bit of searching, I found [the GNOME docs](https://developer.gnome.org/gtk3/stable/gtk-running.html) and they suggested I export the variable `GTK_THEME` to set it, and it worked!

I don't know if this is an issue with Sway or not, but `fcitx` and [japanese input](/blog/2018-08-30-fixing-broken-japanese-fonts-arch-linux) stopped working. All I had to do to fix this was re-install the `fcitx-im` group with pacman.

That's pretty much it!
