---
layout: post
title: DISPLAY – Simple LandTiger NXP LPC1768 video player
tags:
- Embedded
- Landtiger
- Video
- C
- ARM
---

> Based on [original post](http://cas.polito.it/NXP-LANDTIGER@PoliTo-University/?p=225) written for a Computer Architecture university course

# Introduction

I had the opportunity, during the first semester of my first master year at Politecnico di Torino, to get involved in a challenging experience based on developing extra features on a specific embedded system - used in the main course - trying to relax some constraints that are inevitably present on such these limited devices.

The goal of my work was to introduce the capability of (rudimental) video playback on the installed touch-screen display.

The device used is the [LandtTiger LPC1768](https://copperhilltech.com/landtiger-nxp-lpc1768-development-board/) board with the connected [ILI9325](https://cdn-shop.adafruit.com/datasheets/ILI9325.pdf) LCD screen (ILI9320 on some board).

### Expected outcome

During the first meetings with the professor we agreed that the desired behaviour of the application was to display a simple sequence of frames - at a fixed framerate and resolution - loaded somewhere in the device's memory at compilation time.

### Starting point

Luckily, the display comes with a driver library that is in charge of initializing the screen's registers and providing some APIs to display either text or filling single points with the passed color.

However, the available functions were not enough for my purpose since plotting the frame one pixel at a time would have led in unacceptable framerates. So this was absolutely one of the intervention points.

The other main point was to find out a way to first convert than load the short sequence of frames into the board's memory.

Let's dive in.

# Converting the video

First of all I needed to find out a way to convert an input video in a compatible format. In order to do that, the most direct way – and the one I took – is to extract a sequence of frames from it and then print those extracted frames to the screen, with specific delay constraints.

### Extracting frames

For this purpose the choice fell on *ffmpeg* – an extremely powerful tool that converts and edits audio and video streams.

> *ffmpeg* is a quite big tool, used in many other environments. You may want to read the documentation to get better explanation over the used commands

Starting from a regular video file, the two operation to be performed are
  - *spatial cropping* in order to fit the screen size (and to meet the memory requirements as we'll see)
  - *time limitation* in order to extract a portion of the video (same requirements as before)

Translated in *ffmpeg*:

{% highlight bash %}
ffmpeg -i ${INPUT_VIDEO} -vf scale=”-2:${HEIGHT},crop=${WIDTH}:${HEIGHT}” -ss ${START_TIME} -t ${DURATION} “${VIDEO_NAME}_cut.mp4”
{% endhighlight %}

where the non-obvious parameter are *WIDTH* and *HEIGHT* since:

 - in the first part you can omit one of the two (by replacing with a -1 or -2 as appropriate)
 - in the crop part they must be consistent with the scaled down size above specified

Now let’s actually extract the frames:

{% highlight bash %}
ffmpeg -i “${VIDEO_NAME}_cut.mp4” -vf fps=1/${FRAME_OFFSET} -vcodec rawvideo -f rawvideo -pix_fmt rgb565be -f image2 ./${filename}%03d.raw
{% endhighlight %}

I will explain in detail some of the arguments of the previous command:

 - `fps=1/${FRAME_OFFSET}`: “take a frame from source every *FRAME_OFFSET* seconds”
 - `rawvideo`: no header inserted, only pixel representation
 - `-pix_fmt rgb565be`: the flow of data from the Board to the LCD is – in theory, see later – 16-bit based. What we are asking here is “I want every pixel to be represented in RGB 16 bit convention – Big Endian” ([more here](https://en.wikipedia.org/wiki/High_color#16-bit_high_color))
 - output frames will be saved with the .raw extension and with increasing numbers in the filename


At the end of the day, we'll have a folder populated with a certain number or `.raw` files, each representing a single frame of the sequence.

### Image conversion









