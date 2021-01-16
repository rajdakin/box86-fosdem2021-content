---
title: box86
themes:
 - Gaming
website: https://ptitseb.github.io/box86/
logo: stands/box86/logo
description: |
    A linux userspace x86 emulator with a twist

showcase: |
    Discover new possibilties for your RaspberryPI 4 and all other ARM SBC with box86.
    Playing FTL or Into the Breach, Unreal Tournament 99 or 2004, or racing a few laps on Flatout (to name just a few) becomes possible on a small SBC.

new_this_year: |
    <p>
     box86 is targeted towards 32bits.
     While compatibility and speed can be improved, the support of 16bits code (for Wine) is probably the last missing feature for box86.
     <br>
     After that, box64 will be targeted toward 64bits apps.
     It will be a different application, and will allow similar principles with native use of ARM64 native libs directly on x86_64 linux apps.
    </p>

layout: stand
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

Here are 6 videos, the first 2 videos are videos of "Airline Tycoon Deluxe" and "Heretic 2" running on a GigaHertz
OpenPandora (the second one is using the dynarec), and the next 2 videos are videos of "Bit.Trip.Runner" and
"Neverwinter Night" running on an ODroid XU4 (without dynarec), and the last 2 videos are on on a Pi4: Shovel Knight
(video from [@ITotalJustice](https://github.com/ITotalJustice)) and Freedom Planet (video from
[@djazz](https://www.youtube.com/channel/UCoak0PlSYIv9PA-iMqqqa1Q)), also without dynarec.

{{< youtube id="bLt0hMoFDLk" title="Airline Tycoon Deluxe" >}}
{{< youtube id="MM7kWYts7IA" title="Heretic 2 w/ dynarec" >}}
{{< youtube id="8hr71S029Hg" title="Bit.Trip.Runner" >}}
{{< youtube id="B4YN37z3-ws" title="Neverwinter Night" >}}
{{< youtube id="xk8Q30mxqPg" title="Shovel Knight" >}}
{{< youtube id="_QMRMVvYrqU" title="Freedom Planet" >}}

You can find many more box86 video on the [PI Lab Channel](https://www.youtube.com/channel/UCgfQjdc5RceRlTGfuthBs7g).

Compatibility list is there: [https://github.com/ptitSeb/box86-compatibility-list/issues](https://github.com/ptitSeb/box86-compatibility-list/issues)

{{< figure src="icon.png" caption="Logo and Icon made by [@grayduck](https://github.com/grayduck), thanks!" >}}

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
