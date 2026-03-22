---
tags:
  - esp32
  - LCD
  - sdcard
  - microcontrollers
---
# Introduction

After showing images on the CYD board, naturally I was wondering: how can i display larger files?

# Research

I snooped around once again in the [CYD Github Page](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display), and tried the sample code. I had a counterfeit SD card from a previous purchase that I thought could be used, turns out it still can't load properly. Luckily, I had a different SD card lying around and it did work correctly.

![[Pasted image 20260319223624.png|419]]

I removed the SD card to check its contents and indeed there are new files added by the example project.

Cool..... But, what about images...? How do those work?

# Day 1: Finding guides

So I was thinkng: since we are using an SD card, there will be a lot of tutorials on how to display images in the SD card. I was correct, but only partially. All the tutorials boil down to "Trust my code bro", and the worst part is they all use Arduino IDE (which if i were to write natively in PlatformIO it wouldn't work...).

Mainly, I was looking for these things

1. How to read a file from SD card
2. How to read an image from SD card
3. What libraries/utils I need to use to read the image
4. How to display these auxiliary structure on TFT display
5. Any libraries I need to use to display these easily?

# Day 2: Checking out sample codes

After some deliberation, I decided to ask ChatGPT to generate some sample as a reference. You can find the references in [my repository](https://github.com/SpdPnd98/LearnTFTSD.

This was the error message that was produced:

![[Pasted image 20260320081218.png]]

I'm not sure what it means, I'll be researching about BMPs, but according to my understanding, there are 8-bit, 16-bit, 24-bit, 32-bit variants. I don't really know how to differentiate between them yet, but after converting it to 24-bit bmp, I managed to display my first image!

![[IMG_6759.mp4]]

Ok but, this is still kind of a "Trust me bro" approach. Nonetheless, it shows how this approach of content creation has grown outdated in this day and age, as anyone who knows what to ask can ask LLMs for a tutorial. 

However, there are problems with the AI generated code. If you noticed, it slowly loads from bottom to the top, and it looks kind of clunky. From the code, we can find the exact reason for this.

```C++
void drawBMP(const char *filename, int x, int y) {
	...
	read32(bmpFile); // compression
	bmpFile.seek(imageOffset);
	uint8_t r, g, b;
	for (int row = h - 1; row >= 0; row--) {
		for (int col = 0; col < w; col++) {
			b = bmpFile.read();
			g = bmpFile.read();
			r = bmpFile.read();
			tft.drawPixel(x + col, y + row, tft.color565(r, g, b));
		}
	}
	bmpFile.close();
}
```

If you look at this section, it actually reads the RGB values of each pixel in the image, and draws it on the TFT display. I believe there should be a way to read the values and store it somewhere, and offload it to the display all at once? Well, at least ChatGPT says the same thing.

![[Pasted image 20260322151320.png]]

So I decided to look at one of the many video's sample code. Many of the code uses a library called [`JPEGDecoder`](https://github.com/Bodmer/JPEGDecoder/blob/master/examples/Arduino%20TFT/TFT_Jpeg/TFT_Jpeg.ino) and [`Arduino_GFX`](https://github.com/moononournation/Arduino_GFX), which is apparently similar to `Adafruit_GFX` but more generic.

# Day 3: `JPEGDecoder` and `Arduino_GFX`

## Part 1: `JPEGDecoder`

Following the JPEGDecoder sample code, we can see where the gains are found:

```
...
// check if the image block size needs to be changed for the right and bottom edges
    if (mcu_x + mcu_w <= max_x) win_w = mcu_w;
    else win_w = min_w;
    if (mcu_y + mcu_h <= max_y) win_h = mcu_h;
    else win_h = min_h;

    // calculate how many pixels must be drawn
    uint32_t mcu_pixels = win_w * win_h;

    // draw image block if it will fit on the screen
    if ( ( mcu_x + win_w) <= TFTscreen.width() && ( mcu_y + win_h) <= TFTscreen.height()) {
      // open a window onto the screen to paint the pixels into
      //TFTscreen.setAddrWindow(mcu_x, mcu_y, mcu_x + win_w - 1, mcu_y + win_h - 1);
      TFTscreen.setAddrWindow(mcu_x, mcu_y, mcu_x + win_w - 1, mcu_y + win_h - 1);
      // push all the image block pixels to the screen
      while (mcu_pixels--) TFTscreen.pushColor(*pImg++); // Send to TFT 16 bits at a time
    }
    ...

```

Running the sample, we can observe some weird black bars in betwen sections. 
![[IMG_6764.mp4]]

From checking out the code, it is due to the line:
```
TFTscreen.setAddrWindow(mcu_x, mcu_y, mcu_x + win_w - 1, mcu_y + win_h - 1);
```

Basicially, where the image data's start and end was incorrect, but the starting position on the TFT display was correct. The result seems a bit better, but the sliding portion is still visible, the goal is to eliminate them (or make it less perceivable).

## Part 2: `Arduino_GFX` (?)

After compiling time and time again, i could not get it to work.... Perhaps I will look into it in another day. The main culprit is that `Arduino_GFX_Library` couldn't be found.

I tried to include it properly, but there seems to still be issues...

![[Pasted image 20260322164144.png]]

Funnily enough, ChatGPT said that the library only provided better support for calling the images, which if we know how to do it in `JPEGDecoder`, we probably wouldn't need it. So I decided to put GPT's statement to the test.

## Part 3: Only `JPEGDecoder`, but faster?

I asked GPT to generate a code that includes a profiler to see what is causing the time it takes to view the images slow, here are the results

![[Pasted image 20260322195318.png]]

Here is a snapshot of the code
![[Pasted image 20260322195451.png]]So in fact, 3/4 of the time is spent on reading the file, not writing it to the display. This means if we have a more efficient way to read, then we would be able to eliminate the slowness (?). After a quick search, it seems that the reason it takes so long to read the image is because JPEG is meant to be good for compression. So what i was thinking was 2 directions:

1. Store the image as a buffer so that it would be imperceivable (but we'll still feel a lag from point of trigger to image appearing)
2. Store the image as a non-compressed form

## Part 4: The twist?

After wrestling with GPT for hours, and uploading tons of videos that are slow to say the least, I decided to just verify whether I just had a bad board or something. I uploaded the code from [here](https://github.com/thelastoutpostworkshop/esp32-2432S028_video_player) which implemented a playback very nicely, low and behold it just works....

![[IMG_6769.mp4]]

There were some minor issues like fames glitching etc, but i think that can be solved once the images are resized properly, or reducing the screen frequency to something lower.

Suddenly, something clicked in my head: The sample code uses `JPEGDEC`, and my current GPT code uses `JPEGDecode`, could there be a difference?

I quick Google search and I was greeted with this amazing [post](https://atomic14.substack.com/p/the-fastest-esp32-jpeg-decoder) by Chris Greening. Here are the graphs that drew my attention! (Note the images below are Chris's findings, I just took parts that were enlightening to me, if you're interested please give it a read!)

![[Pasted image 20260322234949.png]]

The `JPEGDecoder` library is almost 3 times slower than the `JPEGDEC` library, and the `JPEGDEC` library is by far the more modern one, as it utilizes more RAM to render images faster. `JPEGDecoder` uses much less RAM, hence it renders more slowly.

![[Screenshot 2026-03-22 at 11.51.29 PM.png]]

However, what I don't yet understand is how can a 3x performance boost cause an almost instantaneous feel. That'll likely be the next experimentation I'll do.

Oh, and I also eventually got Arduino_GFX to work properly.
![[Pasted image 20260322235901.png]]

I don't see any difference from previous endeavours, maybe I was mixing up errors while feeling frustrated.
# Conclusion

This project has taken a really big turn, I feel like this is a good stopping point for an initial understanding.

To get an Image from an SD Card to be displayed on the TFT display, we will need to:
1. Use the SD.h library to read the image
2. Use a library like `JPEGDecoder` or `JPEGDEC` to read the images
3. Use a graphical library like `TFT_eSPI` or `Arduino_GFX_Library` to render it onto the screen
4. We can write a wrapper on top of `JPEGDEC` to show a sequence of frames nicely
5. Decoders play a big part in ensuring faster image render speed

While (almost) not perceivable, the mjpeg sequence was indeed longer than the one from [[Using the Yellow Screen]]! That's all I could have hoped for (for now). In the future, I'll dive more deeply into the decoders and graphical libraries.

