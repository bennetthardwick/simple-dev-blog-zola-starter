+++
title = "Fixing broken Japanese fonts on Arch Linux"
description = "How to configure default system fonts and input for working with Japanese on Arch Linux."
aliases = ["/blog/2018-08-30-fixing-broken-japanese-fonts-arch-linux/"]
[taxonomies]
tags = ["archlinux", "beginner", "japanese"]
+++

Setting up fonts for Japanese and other non-latin languages can be a bit difficult on Arch Linux. I find that every time I configure a new system I forget how to make it work perfectly,
and I end up wasting hours trying to find the right answers.

So I'm writing this as a reference in case I ever need to do it again.

## System font configuration

The first issue I'll normally run into is broken fonts. After finishing up an install, non-latin fonts look like this:

![Split the nugget please Dad](./nagetto-watte-touchan.png)

A bit squished. Well this is easily fixed by creating a user font configuration file, at the path `~/.config/fontconfig/fonts.conf`, and filling it with the following:

```xml
<fontconfig>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Noto Serif CJK JP</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans CJK JP</family>
    </prefer>
  </alias>

  <alias>
    <family>monospace</family>
    <prefer>
      <family>Fantasque Sans Mono</family>
      <family>Noto Sans Mono CJK JP</family>
    </prefer>
  </alias>
</fontconfig>
```

It's important to note that I added my default monospace font into the configuration file as well, above the Japanese mono font.

When Arch Linux tries to match a font, it works it's way down through the list of fonts until it finds one that matches.

As the user specified config is one of the first to be matched,
the Noto Sans Mono CJK JP font could be matched quite early, which doesn't look great for English characters.

To combat this, you could not specify a monospace font, or you could edit the system configuration at `/etc/fonts/conf.d/65-nonlatin.conf`.

I find it's a lot easier having the `~/.config/` folder though, since I can sync it with [the rest of my dotfiles](https://github.com/bennetthardwick/dotfiles).

## Input method

Finding a way to write Japanese using Linux is probably one of the harder things to do.

When I have to do it, I normally start by installing `uim`. After spending an hour or so configuring it, I find out that it doesn't work with QT5 apps.

Following some instructions online I'll end up building a version from the AUR but it still won't work.

After a few more hours of searching online, I'll eventually
stumble upon [this reddit post](https://www.reddit.com/r/archlinux/comments/8gw9em/uimanthy_anki/) which will put me on the right track.

### Installing Fcitx

You can find more details about installing `fcitx` at the [Archlinux Wiki](https://wiki.archlinux.org/index.php/Fcitx).

1. Install `fcitx` and `fcitx-mozc` and the config tool.

```
sudo pacman -Sy fcitx-im fcitx-mozc fcitx-configtool
```

2. Set `XMODIFIERS` variables

```
echo "GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx" >> ~/.pam_environment
```

3. Reboot!

After doing those steps, Fcitx and Mozc should both be installed. Go ahead and issue the command `fcitx` to start the daemon.

You can now change up the hotkeys or enable auto-starting, but I prefer only executing it when I need to.
