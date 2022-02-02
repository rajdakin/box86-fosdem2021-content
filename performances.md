---
title: "Performances"
---

## Box86 performances {#perfs}

Like for last year, I will present a series of benchmark to try to show the efficiency (or inefficiency) of both box's
dynarecs. This year, I will benchmark both `box86` and `box64`, to show how `box86` evolved over 1 year of
optimisations, and also show how `box64`, the new 64bits counterpart of `box86` runs.

With `box86` and `box64`, to check efficiency, I used 4 programs: `7z`, `dav1d`, `glmark2`, and `openarena`.
* `7z` is a widly used compression program, available on every distribution, and it comes with an integrated benchmark.
  The benchmark doesn't seem to use much SSE code, and is very CPU performances centric. So it can be used to evaluate
  raw CPU power, and is considered a "worst case scenario" for both box (mostly CPU, no SSE, very little library
  function calls).
* `dav1d` is a video transcoding tools. It use highly optimized SSSE3 or NEON routine. This is the "nightmare
  scenario": with hand-optimized assembly routine and mostly no wrapped function calls.
* `glmark2` on the other hand is used to measure OpenGL performances. Here, CPU is not much used, and it's mostly
  OpenGL calls all along. This is the ideal case scenario for box (mostly native library function calls).
* `openarena` is an open source game, based on idTech3 engine. It include a benchmark mode that will be used here. The
  point of this scenario is to show some "real case usage" of box.

## 7z benchmark {#7z}

### Method {#7z-method}

The `7z` used here is the version 16.02 found in the distribution.
The x86 one comes from Debian, and the native one from the distribution (with the exeption of the Pandora, which was
custom built). I noticed that 32bits ARM version of 7z have higher Decompression result than 64bits ARM64 version,
while compression result are lower (I suspect asm optimisation maybe?).

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

### box86 results {#7z-res86}

| Hardware description | 2021 | Native |    | Emulated |    | Speed % | 2022 | Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Pandora Cortex-a8 1GHz |    | 655 |    | 282 |    | 43% |    | 662 |    | 263 |    | 40% |
| Pyra Cortex-a15 1.5GHz x2 (3) |    | 2510 |    | 976 |    | 39% |    | 2441 |    | 1091 |    | 45% |
| RaspberyPI4 1.5GHz x4 |    | 5733 |    | 2912 |    | 51% |    |    |    |    |    |    |
| ODroid XU4 2Ghz/1.3Ghz x4/4 (1) |    | 3906 |    | 2374 |    | 61% |    |    |    |    |    |    |
| ODroid N2 2Ghz x4/2 |    | ? |    | ? |    | ? |    | 8981 |    | 3811 |    | 42% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 (2) |    | 6912 |    | 3367 |    | 49% |    |    |    |    |    |    |
| FT2000/4 2.6GHz x4 (4) |    | 8280 |    | 4595 |    | 55% |    | 10596 |   | 4916 |    | 46% |
| D2000/8 2.3GHz x8 |    | ? |    | ? |    | ? |    | 18001 |    | 8462 |    | 47% |

\
(1) limiting bench to 4 cores only. \
(2) native was aarch64 in 2021. \
(3) I suspect x86 test triggers some thermal throttle. \
(4) I used a remote control in 2021. I have the physical machine in 2022.

### box64 results {#7z-res64}

| Hardware description | 2022 | Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| RaspberyPI4 1.5GHz x4 |    | ? |    | ? |    | -% |    |
| ODroid N2 2Ghz x4/2 |    | 8883 |    | 3341 |    | 38% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 |    | ? |    | ? |    | -% |    |
| FT2000/4 2.6GHz x4|    | 9977 |   | 4370 |    | 44% |
| D2000/8 2.3GHz x8 |    | 16799 |    | 7572 |    | 45% |    |

### Conclusion {#7z-concl}

So, nothing spectacular on the 7zip case. Emulation speed is more or less the same. In fact, most of the optimisation
have been around x87 and SSE code. Some optimisation around conditional jump is still needed (it was barely started),
and it shows on this cpu-only benchmark. And the picture is the same for 32bits or 64bits emulation.

## dav1d benchmark {#dav1d}

### Method {#dav1d-method}

dav1d is a transcoding application, heavily optimized for SIMD and make use of SSSE on x86 (or SSE4 if availble) and
NEON on ARM. While MMX, SSE and SSE2 convert fairly well to NEON, it's not the case with SSSE3, conversion gets more
complex resulting in emiting 4 or more NEON opcode per SSSE3 one (like `pmaddubsw` on XMM regs generate 10 NEON
opcodes). Also, because the functions of dav1d are hand optimized assembler, the comparison between x86 and native will
give lower number, as the dynarec convert 1:1 opcode, without reordering or removing any opcode. The resuling SSE->NEON
code will be less efficiant than hand optimized routines. The benchmark will use the command
`./dav1d -i Chimera-AV1-8bit-480x270-552kbps.ivf --muxer null`, and just the resulting FPS will be taken. \
When testing multiple thread, add `--framethreads x --tilethreads x` to the command, where the 2 `x` are the number of
threads. The x86 and x86_64 version comes from a Debian distribution.

Example of result:
```
dav1d 0.7.1-91-gba875b9 - by VideoLAN
Decoded 8929/8929 frames (100.0%) - 37.11/1784.02 fps (0.02x)
```
You get '37.11'.

### box86 results {#dav1d-res86}

| Hardware description | 2021 | Native |    | Emulated |    | Speed % | 2022 | Native |   | Emulated |   | Speed % |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Pandora Cortex-a8 1GHz (1) |    | 37.11 |    | 9.43 |    | 25% |   | 37.61 |   | 9.33 |   | 25% |
| Pyra Cortex-a15 1.5GHz x2 1 thread (1) |    | 111.51 |    | 34.90 |    | 31% |   | 120.00 |    | 31.30 |    | 26% |
| Pyra Cortex-a15 1.5GHz x2 2 threads (1) |    | 141.94 |    | 43.97 |    | 31% |    | 146.51 |    | 45.36 |    | 31% |
| RaspberyPI4 1.5GHz x4 1 thread |    | 90.52 |    | 43.18 |    | 48% |    |    |    |    |    |    |
| RaspberyPI4 1.5GHz x4 2 threads |    | 164.53 |    | 84.81 |    | 52% |    |    |    |    |    |    |
| RaspberyPI4 1.5GHz x4 4 threads |    | 189.40 |    | 103.30 |    | 55% |    |    |    |    |    |    |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 1 thread (1)(2) |    | 196.57 |    | 55.80 |    | 28% |    |    |    |    |    |    |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 2 threads (1)(2) |    | 281.97 |    | 58.34 |    | 21% |    |    |    |    |    |    |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 4 threads (1)(2) |    | 374.00 |    | 93.85 |    | 25% |    |    |    |    |    |    |
| ODroid N2 2Ghz x4/2 1 thread (1)(2) |    | ? |    | ? |    | -% |    | 216.26 |    | 57.99 |    | 27% |
| ODroid N2 2Ghz x4/2 2 thread (1)(2) |    | ? |    | ? |    | -% |    | 383.34 |    | 92.40 |    | 24% |
| ODroid N2 2Ghz x4/2 4 thread (1)(2) |    | ? |    | ? |    | -% |    | 500.63 |    | 138.80 |     | 28% |
| FT2000/4 2.6GHz x4 1 thread (2) |    | 290.10 |    | 75.74 |    | 26% |    | 308.61 |    | 80.95 |   | 26% |
| FT2000/4 2.6GHz x4 2 threads (2) |    | 424.93 |    | 128.28 |    | 30% |   | 562.19 |    | 163.44 |    | 29% |
| FT2000/4 2.6GHz x4 4 threads (2) |    | 462.49 |    | 146.87 |    | 32% |   | 658.96 |   | 196.12 |   | 30% |
| D2000/8 2.3GHz x4 1 thread |    | ? |    | ? |    | -% |    | 255.16 |    | 73.25 |    | 29% |
| D2000/8 2.3GHz x4 2 thread |    | ? |    | ? |    | -% |    | 551.93 |    | 165.81 |    | 30% |
| D2000/8 2.3GHz x4 4 thread |    | ? |    | ? |    | -% |    | 811.30 |    | 260.50 |    | 32% |
| D2000/8 2.3GHz x4 8 thread |    | ? |    | ? |    | -% |    | 816.01 |    | 265.34 |    | 33% |

\
(1) native build of dav1d is a custom build, with probably more optimisation than in regular version from distribution. \
(2) native is aarch64 here.

### box64 results {#dav1d-res64}

| Hardware description | 2022 | Native |   |Emulated|    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| RaspberyPI4 1.5GHz x4 1 thread |    | ? |    | ? |    | -% |
| RaspberyPI4 1.5GHz x4 2 threads |    | ? |    | ? |    | -% |
| RaspberyPI4 1.5GHz x4 4 threads |    | ? |    | ? |    | -% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 1 thread |    | ? |    | ? |    | -% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 2 threads |    | ? |    | ? |    | -% |
| Pine64 RockPro64 RK3399 2Ghz/1.5Ghz x2/4 4 threads |    | ? |    | ? |    | -% |
| ODroid N2 2Ghz x4/2 1 thread |    | 216.26 |    | 51.67 |    | 24% |
| ODroid N2 2Ghz x4/2 2 thread |    | 383.34 |    | 89.77 |    | 24% |
| ODroid N2 2Ghz x4/2 4 thread |    | 500.63 |    | 137.60 |    | 27% |
| FT2000/4 2.6GHz x4 1 thread |    | 308.61 |    | ? |    | -% |
| FT2000/4 2.6GHz x4 2 threads |    | 562.19 |    | ? |    | -% |
| FT2000/4 2.6GHz x4 4 threads |    | 658.96 |    | ? |    | -% |
| D2000/8 2.3GHz x4 1 thread |    | 290.57 |    | 65.49 |    | 23% |
| D2000/8 2.3GHz x4 2 thread |    | 636.41 |    | 152.54 |    | 24% |
| D2000/8 2.3GHz x4 4 thread |    | 872.73 |    | 239.47 |    | 27% |
| D2000/8 2.3GHz x4 8 thread |    | 935.60 |    | 246.42 |    | 26% |

## glmark2 benchmark {#glmark2}

### Method {#glmark2-method}

`glmark2` is a program to test OpenGL speed. A version targetting GLES2 hardware also exists, but because GLES2 is not
supported by `box86`, only the Desktop Opengl version is used here. Some distribution (like debian) doesn't include
`glmark2`, so latest version from git source is used here (so 2020.04 version). For platform that doesn't have full
opengl driver, `gl4es` will be used. Note that `glmark2` will crash when run on OpenGL 2.1 only (because it uses
glGenerateMipmap function, but only get that function on OpenGL 3.0+), so `MESA_GL_VERSION_OVERRIDE=3.2` is used on
mesa driver that only expose 2.1 profile, or `LIBGL_GL=30` when using `gl4es`.

The `glmark2` score is simply used here, at default windowed 800x600 resolution (cropped to 800x480 on pandora).
Because Mesa as evolved since last year, I will start fresh and not compared with 2021 result.

### box86 results {#glmark2-res86}

| Hardware description |2022| Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| Pandora Cortex-a8 1GHz gl4es |    | 85 |    | 82 |    | 96% |
| Pyra Cortex-a15 1.5GHz x2 gl4es |    | 284 |    | 280 |    | 98% |
| RaspberyPI4 V3D 4.2 Mesa |    | ? |    | ? |    | -% |
| ODroid XU4 2Ghz x4 gl4es (1) |    | ? |    | ? |    | -% |
| PinePro64 RK3399 2Ghz.1.5Ghz x2.4 mesa |    | ? |    | ? |    | -% |
| ODroid N2 2Ghz x4/2 mesa |    | 733 |    | 720 |    | 98% |
| FT2000/4 2.6GHz x4 Radeon mesa |    | 4540 |    | 4526 |    | 100% |

\
(1) VSync cannot be disabled, lowering results.

### box64 results {#glmark2-res64}

| Hardware description |2022| Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| RaspberyPI4 V3D 4.2 Mesa |    | ? |    | ? |    | -% |
| PinePro64 RK3399 2Ghz.1.5Ghz x2.4 mesa |    | ? |    | ? |    | -% |
| ODroid N2 2Ghz x4/2 mesa |    | 732 |    | 725 |    | 99% |
| FT2000/4 2.6GHz x4 Radeon mesa |    | 4770 |    | 4626 |    | 97% |

\
(1) VSync cannot be disabled, lowering results.

### Conclusion {#glmark2-concl}

When running an app that is mainly relying on external libraries (that are wrapped), near native speeds can be
acheived, with all result here being between 95% and 100% of native speed.

## OpenArena benchmark {#openarena}

This test wasn't present last year. But I did write an article about it
[there](https://box86.org/2021/06/game-performances/).

The benchmark uses some heavy graphics settings, making it quite GPU-limited on low-end machines, so I used an
alternative graphic settings there (no bloom, refelction or shadows), to avoid beeing too GPU-limited, as that would
invalidate the bench.

### box86 results {#openarena-res86}

| Hardware description | 2022 | Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| Pandora Cortex-a8 1GHz gl4es (1) |    | 20.1 |    | ? |    | -% |
| Pyra Cortex-a15 1.5GHz x2 gl4es (1) |    | 25.9 |    | 19.6 |    | 76% |
| RaspberyPI4 V3D 4.2 Mesa |    | ? |    | ? |    | -% |
| PinePro64 RK3399 2Ghz.1.5Ghz x2.4 mesa |    | ? |    | ? |    | -% |
| ODroid N2 2Ghz x4.2 mesa |    | 40.1 |    | 34.3 |    | 86% |
| FT2000/4 2.6GHz x4 Radeon mesa (2) |    | 95.1 |    | 68.1 |    | 72% |
| D2000/8 2.3GHz x8 Radeon mesa |    | 82.0 |    | 60.3 |    | 74% |

\
(1) using a simplified graphic settings (no bloom, reflection or shadows), and fullscreen on device resolution \
(2) at 2560x1440 resolution

### box64 results {#openarena-res64}

| Hardware description | 2022 | Native |    | Emulated |    | Speed % |
| --- | --- | --- | --- | --- | --- | --- |
| RaspberyPI4 V3D 4.2 Mesa |    | ? |    | ? |    | -% |
| PinePro64 RK3399 2Ghz.1.5Ghz x2.4 mesa |    | ? |    | ? |    | -% |
| ODroid N2 2Ghz x4/2 mesa |    | 41.1 |    | 34.8 |    | 85% |
| FT2000/4 2.6GHz x4 Radeon mesa (1) |    | 98.6 |    | 77.4 |    | 78% |
| D2000/8 2.3GHz x8 Radeon mesa |    | 83.9 |    | 67.9 |    | 81% |

\
(1) at 2560x1440 resolution

### Conclusion {#openarena-concl}

On a game scenario, when you have reasonably optimized code and a good amount of function call that can be wrapped, the
performances of both box are between 75% and 85% of the native version, which is satisfying!

## Box86 speed {#speed}

So in real world applications and games, things will be in between the two extreme test cases (`7z` and `dav1d`).

Note that things like SSE opcodes are converted to NEON ones, and most of the time a 1:1 conversion is done, making
optimized SSE/SSE2 codes quite optimal in box86. It's more difficult with SSE3/SSSE3, and converted code gets slower. \
Wrapped functions gets native speeds, and emulated cpu will be at least 50% of what it could be if it was a native app
(or even lower if the app/games contains ASM hand optimized routines). The resulting speed will be something in between
depending of the percentage of emulated code vs wrapped lib ratio...
