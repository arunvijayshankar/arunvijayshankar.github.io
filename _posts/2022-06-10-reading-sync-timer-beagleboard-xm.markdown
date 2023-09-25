---
layout: post
title:  "Reading the System Timer on Beagleboard XM"
author: Arun
tags: [programming, cpp, embedded]
---

One of the exercises in the book 'Embedded Programming with Modern C++ Cookbook' is to read the System Timer on a RaspberryPi. The system timer peripheral on a raspberry pi provides a 64 bit free running counter that increments with every clock tick. Although the book uses QEMU to run a rPi emulator as the target for the programs, I have been using a Beagleboard XM as the target. So far, there have been some changes, but no significant differences in code for the raspberry pi vs the beagleboard. I assumed that reading the system timer would be no different. This was of course, not the case. For a beginner like myself, it was quite challenging to get it working on the bbxm. 

The book starts off by hard coding the base address of the rPi system timer, and a struct to store the two 32 bit sections of the timer:

```
constexpr uint32_t kTimerBase = 0x3F003000;

struct SystemTimer {
  uint32_t CS;
  uint32_t counter_lo;
  uint32_t counter_hi;
};
```

The sync timer address is then mapped using mmap as a shared map and cast as a struct pointer to read the value into counter_lo and counter_hi, which are added together to get the actual timer value. 

```
int memfd = open("/dev/mem", O_RDWR | O_SYNC);

SystemTimer *timer = (SystemTimer*)mmap(NULL, sizeof(SystemTimer),
PROT_READ|PROT_WRITE, MAP_SHARED,
memfd, kTimerBase);
```

I knew that it was unlikely that the timer base address will be the same on the bbxm, so the first challenge was to find the right address. I looked for the AM37 manual online, and it turns out that Texas Instruments will let you download it if you register an account with them. So now I have a TI developer account. Oh, and the manual. The manual listed the Sync Timer base address of the 32 kHz clock as 0x48320000, and that the register was 32 bits long. I made the changes, and compiled the build. But when I ran it, the program would only print out one value

```
System timer: 64
System timer: 64
System timer: 64
System timer: 64
System timer: 64
System timer: 64
```

It looked as if the timer was not updating with every clock tick. My first suspicion was that address was wrong somehow, or that the mapping was not correct. I spent a considerable amount of time trying to get to the bottom of that, all to no avail. The address was as noted in the manual, and there did not seem to be anything wrong with the mapping. So I did the only thing I could think of, and reached out to the good folks on the internet. One person suggested that maybe the timer was not enabled. This seemed like a plausible root cause, so I went back to the manual and started digging to find out how to check if the timer is disabled. I did find the timer controls, but I honestly did not really understand it. But what I did accidentally discover was that even though the base address for the sync timer on the bbxm is 0x48320000, the actual value of the timer is stored at a 0x0010 offset!

So I dutifully changed the address to 0x48320010, re-compiled, and ran it. I hit a Bus Error. What is a bus error now? I asked my browser, which said that it was a hardware error, letting the OS know that CPU cannot access the memory that the process is trying to access. So I was back to some issue with the address. I was convinced that the physical address was correct this time, so maybe there was something wrong with the virtual address? After some more scouring of the internet, I found this [post](https://bakhi.github.io/devmem/). Reading through it, I finally understood that mmap was creating a new mapping in the virtual address space starting at the physical address we specify in the mmap call. For some reason that I don't still understand, providing the physical address + the offset to mmap was returning a bad virtual address. But maybe mapping the timer base address first, and then adding the offset might fix it? I tried this, and it worked! The value updated by ~400 units everytime I ran the binary. I also understood why the program was not updating when I ran it with just the base address. The first 4 bytes of the Sync Timer stored just the timer version. I was just reading this constant over and over again. But after first mapping the base physical address and adding the offset, the program read the actual timer value everytime. Here is the entire program (minus error checking):

```
constexpr uint32_t kTimerBase = 0x48320000;
constexpr uint32_t timer_offset = 0x0010; 


int main() {
    int memfd = open("/dev/mem", O_RDWR | O_SYNC);

    void *map_base = mmap(NULL, sizeof(uint32_t),
    PROT_READ | PROT_WRITE, MAP_SHARED,
    memfd, kTimerBase);

    uint32_t *timer = (uint32_t *)((char *)map_base + timer_offset);

    for (int i = 0; i < 10; i++) {
        std::cout << "System timer: " << *timer;
        std::cout << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
    return 0;
}
```

As an aside, the program as specified in the book, did not run correctly on the rPi emulator either. But once I made the offset changes above to the code, it read the timer value correctly on every run.
