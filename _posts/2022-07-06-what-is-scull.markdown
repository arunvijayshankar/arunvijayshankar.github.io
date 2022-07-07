---
layout: post
title:  "What is scull"
author: Arun
tags: [programming, c, linux, device-drivers]
---

## Scull: Simple Character Utility for Loading Localities

"scull is char driver that acts on a memory area as though it were a device.", is how scull is described in chapter 3 of 
[Linux Device Drivers](https://lwn.net/Kernel/LDD3/). That seemed straightforward enough. Scull's capabilities and allowed operations are described
in detail in the rest of the chapter. Once I finished the chapter, I wanted to learn more about what exactly scull was, and how I could go about
using it. But there seems to be surprisingly little information about it online. I thought I'd write a little something about what I have learned about 
it so far.

Scull, if I understand it correctly, is fundamentally a kernel module. When the module is loaded, using insmod or modprobe, the kernel allocates some 
memory for scull, which then treats that memory region like a character device. Once loaded, scull will show up in /proc/devices:

![](/assets/images/post8/proc_devices.png)

A user can then use it as they would a regular character device. One can open and close it, read from it, or write to it. 

The source code for scull can be found [here](http://gauss.ececs.uc.edu/Courses/c4029/code/drivers/Scull/scull.html). It can be compiled as a module 
by extracting the files from that archive and running `make` at the source directory. Loading and Unloading of the module can be done using scull.load 
and scull.unload respectively. The shell scripts also clean up stale filesystem nodes, and setup new ones after loading the module. The newly created 
nodes will now show up under /dev:

![](/assets/images/post8/ls_dev.png)

## Device types and supported operations

Scull supports different types of devices: 
- scull0 - scull3
  - These are global and persistant, meaning the data stored in them is shared between all the open file descriptors associated with it, and that the data is not lost if a file descriptor is closed.

- scullpipe0 - scullpipe3
  - These are FIFO devices which act as pipes.

- scullsingle, scullpriv, sculluid, scullwuid
  - These are similar to scull0, with limitations on when open can be called on them.


Scull implements the following basic device methods:
- llseek():     Change current read/write position in a file.
- read():       Retrieve data from the device.
- write():      Write data to the device.
- open():       First operation performed on the file.
- ioctl():      Issue device specific commands, which are neither reading or writing.
- release():    Invoked when file structure is released.

## Using scull

To use scull, we have have to first compile it to create scull module. The Makefile in the distribution comes configured, so all that needs to be done
is to run ```sudo make``` from the source directory<sup>*</sup>. You would need a machine running a linux kernel to compile it successfully, I'm pretty sure. Once the module is compiled and ready to be loaded, run the scull.load bash script to load the module, and create the filesystem nodes.

The scull device is now ready to be written to, and read from. The scull distribution comes with a test file: sculltest.c

```
/* sculltest.c
 * A simple example of a C program to test some of the
 * operations of the "/dev/scull" device (a.k.a "scull0"),
 * and the 
 * ($Id: sculltest.c,v 1.1 2010/05/19 20:40:00 baker Exp baker $)
 */
#include <unistd.h>
#include <string.h>
#include <stdio.h>
#include <fcntl.h>

int main() {
   int fd, result, len;
   char buf[10];
   const char *str;
   if ((fd = open("/dev/scull", O_WRONLY)) == -1) {
      perror("1. open failed");
      return -1;
   }

   str = "abcde"; 
   len = strlen(str);
   if ((result = write(fd, str, len)) != len) {
      perror("1. write failed");
      return -1;
   }
   close(fd);

   if ((fd = open("/dev/scull", O_RDONLY)) == -1) {
      perror("2. open failed");
      return -1;
   }
   if ((result = read(fd, &buf, sizeof(buf))) != len) {
      fprintf(stdout, "1. read failed, buf=%s",buf);
      return -1;
   } 
   buf[result] = '\0';
   if (strncmp (buf, str, len)) {
      fprintf (stdout, "failed: read back \"%s\"\n", buf);
   } else {
      fprintf (stdout, "passed\n");
   }
   close(fd);

   ---SNIP---
```
Once the module is loaded, sculltest.c can be compiled and run to test out open, read, and write. Upon a read or write syscall, the corresponding 
function implemented in scull is called. The user can of course, write their own tests to test out supported operations. 

To gain a better understanding of scull and char drivers in general, I implemented a minimalistic version that supports only basic open/close, read/write operations, and only supports the global, persistant scull device type (scull0 - scull3). It can be found [here](https://github.com/arunvijayshankar/vichy). The other major difference is the memory layout. In scull, each device is a linked list of structures, each of which points to a memory area of 4MB at most, though an array of a 1000 pointers, each pointing to a memory area of 4000 bytes. In vichy, each structure in the linked list making up the device points to just one memory area of 4000 bytes. These changes necessitated a few changes in the actual code, but they are small and cosmetic. Which is ok, I think. As a learner, it is a good confidence boost to make small changes to a project and learn by debugging all the issues you run into. This way you can learn the technology, and gain some actual coding experience. 

<sup>*</sup>Note: On newer kernels (I'm running 5.19.0-rc3), scull hits compilations errors, mostly due to a depracated method. The errors can be fixed with these patches
- main.c: https://gist.github.com/arunvijayshankar/213e4dc0442ad3f4cd2b9785abc878a7
- pipe.c: https://gist.github.com/arunvijayshankar/1cd5a2672fe540e196e9c27d163b0407
- access.c: https://gist.github.com/arunvijayshankar/ae12566ac20707eb43afdf3e3d05a570 