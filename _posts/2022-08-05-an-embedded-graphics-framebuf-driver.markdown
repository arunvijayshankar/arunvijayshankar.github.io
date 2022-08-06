---
layout: post
title:  "An embedded graphics framebuf driver for mnemOS"
author: Arun
tags: [programming, embedded, device-drivers, rust]
---

Recently, I wrote a graphics framebuffer driver using the [embedded-graphics crate](https://docs.rs/embedded-graphics/latest/embedded_graphics/). The driver was written for [mnemOS](https://mnemos-dev.jamesmunns.com/book/intro.html), an operating system written in Rust. This contributions marks many firsts for me; first contribution to an OSS project, first piece of embedded software written, first driver written. The driver it self is quite simple. We use [embedded-graphics-simulator](https://docs.rs/embedded-graphics-simulator/latest/embedded_graphics_simulator/) to create a GUI window to display objects/text on. The driver then gives the user a chunk of heap memory onto which an object is drawn. 'Draw' is a trait in embedded-graphics where, given a collection of pixels each with a color and coordinate (the object), each pixel's colour and coordinate (for 8bpp images) is stored in one byte of memory. The entire object can then be stored in a heap array.

![](/assets/images/post9/2022-08-05-21-40-33.png)

Once the object has been drawn onto the chunk of memory, it can then be passed to the e-g-simulator window where it is displayed. However, as the e-g-simulator window only takes a 'SimulatorDisplay' object as an argument, the driver converts the raw bytes into an 'image', which can be passed back to user space and displayed on the window. If we are using a physical display, like a simple OLED display, the raw bytes can be sent over a suitable interface like SPI or I2C. Here is a sample what it looks like on the e-g-simulator:

![](/assets/gifs/Screencast_from_07-27-2022_11_23_56_AM_AdobeExpress.gif)

Contributing to an open source project (or any project, really) has been a goal of mine ever since I started learning programming in earnest. It was really rewarding to go from dreaming about making a contribution, to actually doing it. A small part of me still cannot believe I did it. I owe a huge shoutout to James Munn [(@bitshiftmask)](https://twitter.com/bitshiftmask) for encouraging me to just give it try when I expressed my desire to help out with this issue, but confessed that I wasn't sure if I knew enough to do it. With this under my belt, I feel a lot more confident about my skills, and I am rearing to take on more projects and tasks. Next up is writing a networking driver for mnemOS using smoltcp, a TCP/IP stack for embedded/bare-metal rust projects. I plan to work on this in conjunction with continuing my learning journey in embedded systems. 