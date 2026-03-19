---
tags:
  - "#esp32"
  - "#LCD"
  - "#platformio"
---
# Introduction

I decided to wok on the [Yellow Display](obsidian://open?vault=content&file=2026%2FMarch%2FMysterious%20ESP32%20Yellow%20Screen) to see if it can display anything else other than Hello world. With that, I spun up VSCode + Firefox and started setting up + researching 

# GIFs....?

According to the [documentation](https://doc-tft-espi.readthedocs.io/) of TFT_eSPI, the functions they provide are rather rudimentary, such as drawing pixels on the screen, drawing lines, rectangles etc. If I were to do anything slightly ambitious, it might take too much effort. As such, I have decided to go on a more "unoptimised" path, to just show GIFs instead.

I found this library called [AnimatedGIF](https://github.com/bitbank2/AnimatedGIF) as recommended by the author of TFT_eSPI library to show GIFs

![[Pasted image 20260319090845.png]]

Following his instructions, I went to the AnimatedGIFs and located the sample. For whatever reason, platformIO creates files in cpp, not .ino, so I had to improvise. 

# Arduino to PlatformIO

Essentially, I copied the content of the files `TFT_eSPI_memory.ino` and `GIFDraw.ino` into `main.cpp` and `GIFDraw.cpp`, I then added a header called `GIFDraw.h` which contains the function definition of `GIFDraw`. There was also a need to add the other standard imports such as `#include <stdint.h>` for `uint_t` to be found properly.

In the comments, they mentioned that the variable `tft` doesn't need to be defined as it will be imported into `TFT_eSPI_memory`, but it doesn't compile on my side. As such, i added an `extern TFT_eSPI tft;` line to ensure it is able to find the corresponding variable.

After compiling, I was able to upload as well! Cheers!

![[IMG_6755.mp4]]

# Customising...

Well, I dunno about you, but I'm neutral to the badgers (oops), so I wanted to change the GIF that is shown. After checking out the documentation of `AnimatedGIF`, I eventually stumbled upon another tool that `bitbank2` wrote called [`image_to_c`](https://github.com/bitbank2/image_to_c), which converts a given gif to exactly what is required (except for including the stdint header manually). 

To use it, simply add the generated file into the `header` folder, include it in your program and it should work! 

...Is what I hoped to tell you haha. See, ESP32 has some memory constraints. Luckily for us there is a SD card module on this CYD. But I'll explore that another day. For now, I just cut my GIF shorter so that it runs.

![[IMG_6756.mp4]]

# Conclusion
Looks like there are limitations on the ESP32 CYD, althought I'm quite surprised seeing it run so smoothly. Throughout writing this section, I have also left the screen on and I'm happy to say it isn't hot to the touch, it isn't warm either (more like barely warm?). Excited to see what I can do with this board!