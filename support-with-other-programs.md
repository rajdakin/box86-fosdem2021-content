---
title: "Support with other programs"
---

## Unity game emulation {#unity}

[TODO: check for updates, update for box64]

Running Unity games is a hit or miss for now. Unity uses Mono (which uses signals that are not well emulated enough),
and a runtime embedded in the main binary. A solution would be to use a native version of the libmono library used by
Unity (it can be found [here](https://github.com/Unity-Technologies/mono) and needs to be built from source). But the
wrapping of this lib is tricky, and not done for now, so the only solution is to emulate everything. The tricky part is
to emulate the "JIT" code emitted by Mono, however with the new "protected memory" mechanism implemented it should be
running with correct performance now.

You should also note that some Unity3D games require OpenGL 3+ which can be tricky to provide on ARM SBC (single-board
computers) for now.

**TL;DR:** Not all Unity games work and can require a high OpenGL profile, but the speed, for the ones running, should
be correct now.

## Steam {#steam}

[TODO: check for updates, update for box64]

Linux Steam's can run now with box86. But it's still a bit unstable, and not everything works. First problem is steam
crashing after the 'Sign in' window, if you encounter this issue, you may need to add libappindicator. Installing the
package on debian `sudo apt install libappindicator1`.

Once open, Steam will only work on "Small Mode" and in "Big Picture", not in the regular "Large Mode" (this is because
some steam component used in the browser view are only 64 bits now). So go in the "View" menu and switch to Small view,
else the list will stay empty.

Final words, to avoid the "libc.so.6 is absent" message, you can use `STEAMOS=1` and `STEAM_RUNTIME=1` as environment
variables.

(Steam for Windows installs fine but doesn't work yet)

Some Steam games (most Source engine games, like "Portal" or "Half-Life 2") use libtcmalloc. box86 will detect it and
will try to LD_PRELOAD it, for better compatibility. While it should work without the aformentionned feature, it is
safer to add it to your system if you intend to play those game (something like `sudo apt install libtcmalloc-minimal4`
to install it on Debian and Debian's deriatives).

## Wine {#wine}

[TODO: check for updates, update for box64]

Wine is now partly supported. Wine integrated program all runs, and some windows programs and games now runs fine.
Don't forget most Windows games use Direct3D, this may require a complete OpenGL driver and as high profile as possible
(and gl4es with ES2 backend have issues with Wine for now). Also, Vulkan is wrapped but mostly untested on box86
(tested on Pi4 and it doesn't work because of the unstability of the Vulkan driver itself).

*Note:* if you plan to use box86 with Wine on Raspberry Pi 3 or earlier, those models use a default OS that have a
kernel with a 2/2 Split (meaning 2G of space for user program, and 2G of space for the Kernel). This is not compatible
with Wine programs that needs to access memory > 2Gb address. So you'll need to reconfigure your kernel for a 3G/1G
split. Use your favorite search engine for instructions on how to do that.

## Other programs {#others}

See also this [compatibility list](https://box86.org/app/).
