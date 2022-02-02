---
title: "Performances"
---

# Box86 performances {#perfs}

[TODO: update for 2022, data is from last Fosdem]

How to measure an emulator performances? Because it doesn't run on itself, a raw performance measurement is not
possible. What is possible is comparing how 2 builds of the same program perform: one native and one on the emulated
architecture. While compilation options and efficiency of the compiler on targetted architecture may still introduce
unknown variables, it can give a good base for comparison.

With `box86`, to check efficiency, I used 3 programs: `7z`, `dav1d` and `glmark2`.
* `7z` is a widly used compression program, available on every distribution, and it comes with an integrated benchmark.
  The benchmark doesn't seem to use much SSE code, and is very CPU performances centric. So it can be used to evaluate
  raw CPU power, and is considered a "worst case scenario" for `box86` (mostly CPU, no SSE, very little library
  function calls).
* `dav1d` is a video transcoding tools. It use highly optimized SSSE3 or NEON routine. This is the "nightmare
  scenario": with hand-optimized assembly routine and mostly no wrapped function calls.
* `glmark2` on the other hand is used to measure OpenGL performances. Here, CPU is not much used, and it's mostly
  OpenGL calls all along. This is the ideal case scenario for `box86` (mostly native library function calls).

# 7z benchmark {#7z}

## Method {#7z-method}

The `7z` used here is the version 16.02 found in the distribution.
The x86 one comes from Debian, and the native one from the distribution (with the exeption of the Pandora, which was
custom built).

The command `7z b` was used, the output looks like that:
```
7-Zip [32] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,32 bits,4 CPUs LE)

LE
CPU Freq:   792  1486  1498  1495  1498  1492  1496  1493  1496

RAM size:    3776 MB,  # CPU hardware threads:   4
RAM usage:    882 MB,  # Benchmark threads:      4

                       Compressing  |                  Decompressing
Dict     Speed Usage    R/U Rating  |      Speed Usage    R/U Rating
         KiB/s     %   MIPS   MIPS  |      KiB/s     %   MIPS   MIPS

22:       3639   333   1062   3540  |      92329   396   1989   7877
23:       3539   338   1068   3606  |      90082   394   1976   7794
24:       3585   359   1075   3855  |      87605   395   1945   7691
25:       3506   367   1090   4004  |      84190   394   1901   7493
----------------------------------  | ------------------------------
Avr:             349   1074   3751  |              395   1953   7714
Tot:             372   1513   5733
```

I'll report only the last number (so here 5733), and compare, per hardware, between native and emulated program.

## Result {#7z-res}

| Hardware description |    | Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| Pandora Cortex-a8 1GHz |    | 655 |    | 282 |    | 43% |
| Pyra Cortex-a15 1.5GHz x2 (3) |    | 2510 |    | 976 |    | 39% |
| RaspberyPI4 1.5GHz x4 |    | 5733 |    | 2912 |    | 51% |
| ODroid XU4 2Ghz/1.3Ghz x4/4 (1) |    | 3906 |    | 2374 |    | 61% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 (2) |    | 6912 |    | 3367 |    | 49% |
| FT2000/4 2.6GHz x4 (2)(4) |    | 8280 |    | 4595 |    | 55% |

\
(1) limiting bench to 4 cores only. \
(2) native is aarch64 here. \
(3) I suspect x86 test trigger some thermal throttle. \
(4) the use of a remote control software certainly lower the results.

## Conclusion {#7z-concl}

On a CPU intensive app, without SSE3+ optimizations that uses flags and conditional jumps a lot (that's the most
expansive operations), the emulated software is roughly 50% of the native speed. While this number can still be
improved with the current dynarec design (I have a few more optimizations ideas), this probably won't change much in
the future (it will probably top at maybe 60% later if the last optimization round works fine).

# dav1d benchmark {#dav1d}

## Method {#dav1d-method}

dav1d is a transcoding application, heavily optimized for SIMD and make use of SSSE on x86 (or SSE4 if availble) and
NEON on ARM. While MMX, SSE and SSE2 convert fairly well to NEON, it's not the case with SSSE3, conversion gets more
complex resulting in emiting 4 or more NEON opcode per SSSE3 one (like `pmaddubsw` on XMM regs generate 10 NEON
opcodes). Also, because the functions of dav1d are hand optimized assembler, the comparison between x86 and native will
give lower number, as the dynarec convert 1:1 opcode, without reordering or removing any opcode. The resuling SSE->NEON
code will be less efficiant than hand optimized routines. The benchmark will use the command
`./dav1d -i Chimera-AV1-8bit-480x270-552kbps.ivf --muxer null`, and just the resulting FPS will be taken. \
When testing multiple thread, add `--framethreads x --tilethreads x` to the command, where the 2 `x` are the number of
threads. The x86 version comes from a Debian distribution.

Example of result:
```
dav1d 0.7.1-91-gba875b9 - by VideoLAN
Decoded 8929/8929 frames (100.0%) - 37.11/1784.02 fps (0.02x)
```
get 37.11

## Result {#dav1d-res}

| Hardware description |    | Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| Pandora Cortex-a8 1GHz (1) |    | 37.11 |    | 9.43 |    | 25% |
| Pyra Cortex-a15 1.5GHz x2 1 thread (1) |    | 111.51 |    | 34.90 |    | 31% |
| Pyra Cortex-a15 1.5GHz x2 2 threads (1) |    | 141.94 |    | 43.97 |    | 31% |
| RaspberyPI4 1.5GHz x4 1 thread |    | 90.52 |    | 43.18 |    | 48% |
| RaspberyPI4 1.5GHz x4 2 threads |    | 164.53 |    | 84.81 |    | 52% |
| RaspberyPI4 1.5GHz x4 4 threads |    | 189.40 |    | 103.30 |    | 55% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 1 thread (1)(2) |    | 196.57 |    | 55.80 |    | 28% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 2 threads (1)(2) |    | 281.97 |    | 58.34 |    | 21% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 4 threads (1)(2) |    | 374.00 |    | 93.85 |    | 25% |
| FT2000/4 2.6GHz x4 1 thread (1)(2) |    | 290.10 |    | 75.74 |    | 26% |
| FT2000/4 2.6GHz x4 2 threads (1)(2) |    | 424.93 |    | 128.28 |    | 30% |
| FT2000/4 2.6GHz x4 4 threads (1)(2) |    | 462.49 |    | 146.87 |    | 32% |

\
(1) native build of dav1d is custom build, with probably more optimisation than in regular version from distribution. \
(2) native is aarch64 here.

# glmark2 benchmark {#glmark2}

## Method {#glmark2-method}

`glmark2` is a program to test OpenGL speed. A version targetting GLES2 hardware also exists, but because GLES2 is not
supported by `box86`, only the Desktop Opengl version is used here. Some distribution (like debian) doesn't include
`glmark2`, so latest version from git source is used here (so 2020.04 version). For platform that doesn't have full
opengl driver, `gl4es` will be used. Note that `glmark2` will crash when run on OpenGL 2.1 only (because it uses
glGenerateMipmap function, but only get that function on OpenGL 3.0+), so `MESA_GL_VERSION_OVERRIDE=3.2` is used on
mesa driver that only expose 2.1 profile, or `LIBGL_GL=30` when using `gl4es`.

The `glmark2` score is simply used here, at default windowed 800x600 resolution (cropped to 800x480 on pandora).

## Result {#glmark2-res}

| Hardware description |    | Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| Pandora Cortex-a8 1GHz gl4es |    | 85 |    | 84 |    | 99% |
| Pyra Cortex-a15 1.5GHz x2 gl4es |    | 266 |    | 258 |    | 97% |
| RaspberyPI4 V3D 4.2 Mesa 20.3 |    | 110 |    | 110 |    | 100% |
| RaspberyPI4 V3D 4.2 Mesa 20.2 |    | 122 |    | 120 |    | 98% |
| ODroid XU4 2Ghz x4 gl4es (1) |    | 54 |    | 54 |    | 100% |
| PinePro64 RK3399 2Ghz.1.5Ghz x2.4 mesa 21.1 (2) |    | 577 |    | 565 |    | 98% |
| FT2000/4 2.6GHz x4 Radeon mesa 18.3 (2)(3) |    | 1149 |    | 1093 |    | 95% |

\
(1) VSync cannot be disabled, lowering results. \
(2) native is aarch64 here. \
(3) the use of a remote control software certainly lower the results

## Conclusion {#glmark2-concl}

When running an app that is mainly relying on external libraries (that are wrapped), near native speeds can be
acheived, with all result here being between 95% and 100% of native speed.

# Box86 speed {#speed}

So in real world applications and games, things will be in between those two extreme test cases.

Note that things like SSE opcodes are converted to NEON ones, and most of the time a 1:1 conversion is done, making
optimized SSE/SSE2 codes quite optimal in box86. It's more difficult with SSE3/SSSE3, and converted code gets slower. \
Wrapped functions gets native speeds, and emulated cpu will be at least 50% of what it could be if it was a native app
(or even lower if the app/games contains ASM hand optimized routines). The resulting speed will be something in between
depending of the percentage of emulated code vs wrapped lib ratio...
