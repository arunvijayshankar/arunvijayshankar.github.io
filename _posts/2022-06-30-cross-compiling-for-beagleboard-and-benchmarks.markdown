---
layout: post
title:  "Cross Compiling for BeagleBoard targets, and benchmarks"
author: Arun
tags: [programming, c, cpp, embedded]
---

I have been using a BeagleBoard-xM as a target for the recepies in the book Embedded Programming with Modern C++ Cookbook, and here are a few of the interesting things I've come across:

- ## Cross Compiler
The book uses a Raspberry Pi emulator on QEMU as a target, and uses the gcc/g++ arm cross compiler (arm-linux-gnueabi-gcc/g++) to build the binaries. As the RPi and BeagleBoard both run arm processors, I assumed that a binary that runs on Raspberry Pi would work on BeagleBoard as well. Spoiler alert: a binary compiled to run on an RPi, will not run on BeagleBoard. Google-fu helped me understand that the BeagleBoard needs a slightly different cross-compiler: arm-linux-gnueabi**hf**-gcc/g++. Once I changed the Cmake file accordingly, I could run the program on BeagleBoard.

- ## Uninitialized Pointers
A small program that would create a shared memory region and then have two processes read from, and write to it, gave me unexpected results (and sometimes hit segmentation faults on the last read) on the beagleboard and RPi, whereas on the build system it crashed immediately. It turned out that I was derefencing an uninitialized pointer. On the build system the offending pointer was set to 0x0, but on the bbxm and raspberry pi, it was set to a random pointer. I'm not sure if this is something to do with the architecture or the compiler. 

- ## System Timer
One of the exercises involved writing a program to read the system timer on the rpi. This was a little bit of a challenge to port to the BBxM as the beagleboard's system timer was structured a little differently. I wrote about this in more detail [here](https://arunvijayshankar.github.io/archive/reading-sync-timer-beagleboard-xm/)

- ## Benchmarks
Finally I ran lmbench on the build system, an x86_64 machine running ubuntu, and on the beagleboard, which is armv7 running debian:

BeagleBoard-xM benchmarks

![BeagleBoard-xM Benchmarks](/assets/images/post7/bbxm_bench.png)

Build system benchmarks

![Build system (x86_64) Benchmarks](/assets/images/post7/x86_bench.png)
