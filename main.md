---
title: "About"
---

## Linux Userspace x86 and x86_64 Emulators with a twist {#title}

This project now has a website, available [here](https://box86.org) and a nice compatibility list
[there](https://box86.org/app/).

box86 and box64 lets you run x86 and x86_64 Linux programs respectively (such as games) on non-x86 Linux, like ARM (the host system
needs to be/emulate little-endian 32bit and 64bits respectively).

You *NEED* a 32-bit subsystem to run and build box86. box86 is useless on 64-bit only systems. Also, you *NEED* a
32-bit toolchain to build box86. A toolchain that only support 64-bit will not compile box86, and you'll get errors
(typically on AArch64, you get "-marm" not recognized, and you'll need a multiarch or chroot environnement).

Because box86/box64 uses the native versions of some "system" libraries, like libc, libm, SDL, and OpenGL, it's easy to
integrate and use, and performance can be surprisingly high in some cases.

Most x86/x86_64 games need OpenGL, so on ARM platforms, you may need a solution like
[gl4es](https://github.com/ptitSeb/gl4es), as some (older) ARM platforms only support OpenGL ES and/or their OpenGL
implementation is dodgy.

box86 and box64 also integrate a DynaRec (dynamic recompiler) for the ARM platform, providing a speed boost between 5
to 10 times faster than only using the interpreter.

Many games already work, for example: WorldOfGoo, Airline Tycoon Deluxe, and FTL. Many of the GameMaker Linux games
also run fine (there is a long list, among which are UNDERTALE, A Risk of Rain, and Cook Serve Delicious).
Windows games, like Flatout2 or Crysis, can also be run (depending on the hardware of course) using Wine.

[Here is a page listing some performance tests on box86 and box64.]({{< relref "performances.md" >}})

You can find many more box86/box64 videos on the
[MicroLinux](https://www.youtube.com/channel/UCwFQAEj1lp3out4n7BeBatQ),
[Pi Labs](https://www.youtube.com/channel/UCgfQjdc5RceRlTGfuthBs7g) or
[The Byteman](https://www.youtube.com/channel/UCEr8lpIJ3B5Ctc5BvcOHSnA) YouTube channels.

The compatibility list is there: [https://box86.org/app/](https://box86.org/app/)
(data is requested at [this repo](https://github.com/ptitSeb/box86-compatibility-list/issues)).

{{< figure src="/stands/box86/icon86.png" caption="Logo and icon made by [@grayduck](https://github.com/grayduck), thanks!" width="86px" >}}
{{< figure src="/stands/box86/icon64.png" caption="Logo and icon made by [@grayduck](https://github.com/grayduck), thanks!" width="86px" >}}

Note that this project is not to be mistaken with [86box](https://github.com/86Box/86Box), a nice "Full system"
emulator specialized in early (to fairly recent) PC hardware.

<!-- In-place HTML -->
<video width="720" height="90" controls>
	<source src="https://video.fosdem.org/2022/stands/box86/box86_video1.mp4" type="video/mp4">
	<source src="https://video.fosdem.org/2022/stands/box86/box86_video1.webm" type="video/webm">
	Your browser does not support the video tag.
</video>

## Notes about 64-bit platforms {#about-64bits}

Because box86 works by directly translating function calls from x86 to host system, the host system (the one box86 is
running on) needs to have 32-bit libraries. box86 doesn't include any 32-bit <-> 64-bit translation. So basically, to
run box86 for example on an ARM64 platform, you will need to build box86 for ARM 32-bit, and also need to have a
chroot with 32-bit libraries.

Also note that box64 is only able to run x86_64 binaries on 64-bit platforms. You will still need box86 (and a 32-bit
chroot) to run x86 binaries (in fact, the same is the case on actual x86_64 Linux).

## Compatibilities with other programs {#compat-w-other-programs}

See [the dedicated page]({{< relref "support-with-other-programs.md" >}}).

## Final words {#final-words}

If you use box86 or box64 in your project, please don't forget to mention it.
