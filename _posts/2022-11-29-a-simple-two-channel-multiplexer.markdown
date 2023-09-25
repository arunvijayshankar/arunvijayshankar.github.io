---
layout: post
title:  "A simple two channel multiplexer"
author: Arun
tags: [programming, digital-circuits]
---

A couple of weeks ago, I came across this equation for a two channel multiplexer in a book on x86 Assembly 

```
Z = ( Y & S ) | ( X & ~S )
```

I was a little surprised at how simple the equation was and, out of curiosity, I drew a quick schematic of it

![](/assets/images/post10/mplexer_schem.png)

It occured to me that I could build this for a simple and fun project. I had never really built any circuits before, and this seemed like a good first attempt. So I ordered the logic gates, and put it together once they were delivered, a few days later. It's a very simple circuit and it was really easy to build.

![](/assets/images/post10/mplexer_pic.png)

I used my beaglebone board to power the circuit, and for the input data streams for the two channels on the multiplexer. For the inputs I chose two pwm signals of 2Hz and 5Hz respectively and forwarded them from GPIO header pins on the beaglebone black. I kept the pwm signal frequency low to make the channel selection easier to observe. The circuit uses a push-button to flip the selection channel to choose between the two input channels. Here is a small demo of the circuit in action.

![](/assets/images/post10/demo_final_gif.gif)


