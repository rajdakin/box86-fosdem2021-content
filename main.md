---
title: "About"
---

## Linux Userspace x86 Emulator with a twist {#title}

box86 lets you run x86 Linux programs (such as games) on non-x86 Linux, like ARM (host system needs to be 32bit little-endian).

You *NEED* a 32-bit subsystem to run and build box86. box86 is useless on 64-bit only systems. Also, you *NEED* a
32-bit toolchain to build box86. A toolchain that only support 64-bit will not compile box86, and you'll get errors
(typically on aarch64, you get "-marm" not recognized, and you'll need a multiarch or chroot environnement).

Because box86 uses the native versions of some "system" libraries, like libc, libm, SDL, and OpenGL, it's easy to
integrate and use, and performance can be surprisingly high in some cases.

Most x86 games need OpenGL, so on ARM platforms a solution like [gl4es](https://github.com/ptitSeb/gl4es) is usually
necessary. (Most ARM platforms only support OpenGL ES and/or their OpenGL implementation is dodgy (see OpenGL on
Android).)

box86 now integrates a DynaRec (dynamic recompiler) for the ARM platform, providing a speed boost between 5 to 10 times
faster than only using the interpreter.

Many games already work, for example: WorldOfGoo, Airline Tycoon Deluxe, and FTL. Many of the GameMaker Linux games
also run fine. (there's a long list, among them are UNDERTALE, A Risk of Rain, and Cook Serve Delicious)

[Here is a page listing two performance tests on box86.]({{ relref "performances.md" }})

You can find many more box86 video on the [PI Lab Channel](https://www.youtube.com/channel/UCgfQjdc5RceRlTGfuthBs7g).

Compatibility list is there: [https://github.com/ptitSeb/box86-compatibility-list/issues](https://github.com/ptitSeb/box86-compatibility-list/issues)

{{< figure src="/stands/box86/icon.png" caption="Logo and icon made by [@grayduck](https://github.com/grayduck), thanks!" width="86px" >}}

Note that this project is not to be mistaken with [86box](https://github.com/86Box/86Box), a nice "Full system"
emulator specialized in early (to fairly recent) PC hardware.

## Notes about 64-bit platforms {#about-64bits}

Because box86 works by directly translating function calls from x86 to host system, the host system (the one box86 is
running on) needs to have 32-bit libraries. box86 doesn't include any 32-bit <-> 64-bit translation. So basically, to
run box86 for example on an ARM64 platform, you will need to build box86 for ARM 32-bit, and also need to have a
chroot with 32-bit libraries.

Also note that, even if, on day, there is a box86_64, this one will only be able to run x86_64 binaries on 64-bit
platforms. You will still need box86 (and a 32-bit chroot) to run x86 binaries (in fact, the same is the case on actual
x86_64 Linux).

## Compatibilities with other programs {#compat-w-other-programs}

See [the dedicated page]({{< relref "support-with-other-programs.md" >}}).

## Final words {#final-words}

If you use box86 in your project, please don't forget to mention box86.
