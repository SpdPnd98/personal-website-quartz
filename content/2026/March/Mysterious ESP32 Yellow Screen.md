---
tags:
  - "#esp32"
  - "#microcontrollers"
  - "#LCD"
---
| Back                | Front               |
| ------------------- | ------------------- |
| ![[IMG_6753 1.jpg]] | ![[IMG_6752 1.jpg]] |
# Introduction
Quite a while ago, I have purchased this LCD screen + ESP32 combo, it seemed pretty interesting because I wouldn't need to wire it myself, and it comes in a neat package. I bought it off Lazada. 

Although I am a [Fabacademy graduate](https://fabacademy.org/students/alumni-list.html#_2022), I dare not claim to be an expert in hardware, although I would like to think I know enough to figure out how this works.

![[Pasted image 20260317202523.png]]

# Documentation...?

Hilariously enough, I did not check whether the seller provide any documentation. And to my horror, they gave a documentation on how to run the sample source code without providing the actual source code (???)

![[Screenshot 2026-03-17 at 8.30.03 PM.png]]

And hence began my journey to find the documentation for this mysterious looking board...

Surprisingly, a quick search got me [this page](https://www.elecrow.com/wiki/2.8_240x320_ESP32_LCD_Touch_Display_With_WiFi_and_BTBLE.html) which looked very promising:

![[Pasted image 20260317203801.png]]

Initially I was rather happy about it, but after reading it more thoroughly, something felt off as the layout of the board seems rather different. This got me into the rabbit hole of find all sorts of "Yellow boards" for this ESP32's screen (as well as variants of each of them). I was baffled to say the least, but I also half-expected this as well since it's a "no-name board" after all. 

I was also surprised by the number of variants that are so close, yet so different from each other, each of them having names and layouts that are distinct enough to differentiate between them.

Eventually, I stumbled upon this [Github page](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display) which had the exact layout as my board. They even had a flowchart to determine whether my module is what is on their page. 

![[Pasted image 20260317203408.png]]
So yes, it is a CYD (whatever that is i guess....). And it uses the [TFT-eSPI](https://github.com/Bodmer/TFT_eSPI) library. Finally, a lead!

Edit: Yea.... as long as its yellow it is a type of CYD haha

# Setting up + Testing

To setup, what I did was install Platform.io on my VSCode instance. You can find it from the extensions tab

![[Pasted image 20260317204400.png]]

Next, I cloned the repo above, and opened the [1-Hello World example](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display/blob/main/Examples/Basics/1-HelloWorld/1-HelloWorld.ino) and tried to flash to the device.

Here's the first issue I faced, there was some kind of error while uploading...?

![[Pasted image 20260317205052.png]]

After a bit of investigation, apparently sometimes it wouldn't work with a baud rate of 921600, so it was suggested on the Github repo to use baud rate of 115200. That worked!

![[Pasted image 20260317210015.png]]

Here is an image of the sample code running.

![[IMG_6754.jpg]]

# Conclusion

The mysterious yellow board has been decoded, as it was actually a CYD board. Though I'm still unsure what that means, at least I now know that this particular board is controllable via the TFT_eSPI library. 

The second lesson is: please buy boards that are well documented! But then again, if you're confident that you are able to figure it out, should you even? :P

Edit: I realized CYD stands for "Cheap Yellow Display", which definitely clarifies something for some people i guess haha.