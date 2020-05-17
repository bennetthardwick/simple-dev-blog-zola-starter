+++
title = "Speeding up Arch Linux AUR build times"
description = "Compiling packages from the AUR can take forever. You can speed them up dramatically by parallelizing the build."
aliases = ["/blog/2018-04-17-speeding-up-archlinux-aur-builds/"]
[taxonomies]
tags = ["archlinux", "beginner"]
+++

I have a T440s that runs Arch Linux. It's the computer that I use at University and I spend a lot of time programming on it. One of the things that I hate about it however, is the amount of time it takes to build a package from the AUR.

## Activate Turbo

One of the reasons things aren't compiling as fast as they should is because the `make` part of the build is only running on one core. Actually, it's because all the jobs that `make` performs are running one after the other. To get `make` to use more cores, you have to set it up to run parallel jobs. This is an example of a `PKGBUILD` file. The `...` means that lines have been omitted.

```
...
build() {
...
cmake -j6 -G "Unix Makefiles" ../ \
...
make -j6
...
```

The `-j6` option for `cmake` and `make` means that it will run up to 6 parallel jobs. Whilst my computer only has 4 cores, it's best practice to run 1.5x more jobs than the amount of cores / hardware threads you have at your disposal. This is because whenever a job stops to wait for IO, there will still be something for the core to do while it's waiting.

Unfortunately, in my short time of searching, I haven't found a global solution, but to be honest I'm not too upset with that. I've been searching for a reason to inspect the `PKBUILD` files, and I think doing this will give me a better understanding of what's really being installed on my system.

Thanks to [this blog post](https://blog.kitware.com/cmake-building-with-all-your-cores/) for helping me figure this out!
