---
title: Prerendering Music Visualizations for YouTube
date: 2018-06-15 15:09:43
tags:
---

<div class="aspect-ratio sixteen-nine"><iframe width="640" height="390" src="//www.youtube.com/embed/TyhisYKU2Dg" frameborder="0" allowfullscreen="" style="margin-bottom: 20px;"></iframe></div>

Recently, I came across the [butterchurn](https://github.com/jberg/butterchurn) JavaScript library, which brings the classic [Milkdrop](https://en.wikipedia.org/wiki/MilkDrop) music visualizer to WebGL. I'd also been talking to a friend about his recently released [album](https://cityofforests.bandcamp.com/releases), and wondered if I could figure out how to render a video of Milkdrop to a track from the album and upload it to YouTube.

<!-- more -->

Butterchurn is designed to render frames to a `<canvas>` HTML element, but I wanted to get the output into a video file on disk that I'd be able to upload to YouTube. Some research turned up [CCapture.js](https://github.com/spite/ccapture.js/), a library for capturing `<canvas>` contents and exporting to a variety of formats. I quickly discovered that exporting each frame in real time wasn't going to work; the frame captures couldn't keep up with the music, even at low resolution.

Butterchurn samples the audio input around 60 times per second (using `requestAnimationFrame`), calculates a fast Fourier transform (FFT) of the audio signal at each point, then renders a frame. To fix the stuttering problem, I modified butterchurn to separate the sampling process from the rendering process.  The sampling pass could now proceed at real time, with the expensive rendering pass done afterwards.

After a sampling pass completed, I could iterate over the generated collection of FFT samples and use butterchurn to render each frame from memory instead of taking another live sample. For each frame, I had CCapture.js send the PNG data over a websocket to a local NodeJS server, which saved the frames to disk and then used ffmpeg to stitch all the frames together into an mp4 video. So far so good, except the resulting video didn't match up correctly with the audio!

I realized that butterchurn's sample rate was not a consistent 60 times per second, and actually varied pretty widely. Fortunately, each FFT sample I was grabbing also included some other information, including a timestamp indicating how far into the track it was captured. Armed with this information, I found [mp4fpsmod](https://github.com/nu774/mp4fpsmod), which would allow me to modify my 60 fps constant frame rate (CFR) video into a variable frame rate video (VFR), with each frame's position indicated by the timestamp of the sample I used to render it.

The final step was to use [ffmpeg](https://en.wikipedia.org/wiki/FFmpeg) to transcode the video one more time, to add the audio track and to convert it back into CFR, approximating the VFR clip I created in the previous step. This happens by dropping and duplicating frames as needed.

The easiest part of the whole process was just dragging and dropping the video onto YouTube. The link above shows the output of a 2560x1440, 60fps rendering.

Since the rendering can take as long as you want, you can target any output resolution, assuming you have enough hard drive space to store the individual PNG frames. The render above took about 100GB of temp space.

If you'd like to try this out yourself, please note that my goal was to hack my way to a working solution as quickly as possible so the code is quite ugly :).

You'll need:

- [milkdrop-prerenderer](https://github.com/markneub/milkdrop-prerenderer)
- [butterchurn](https://github.com/markneub/butterchurn)
- [CCapture.js](https://github.com/spite/ccapture.js/)
- [ffmpegserver.js](https://github.com/greggman/ffmpegserver.js)
- [mp4fpsmod](https://github.com/nu774/mp4fpsmod)
- ffmpeg (maybe already installed on your machine)

Once you've rendered the video the first time, you'll need to use mp4fps mod to correct for the variable framerate. This requires getting the timestamps from the sampling phase into a timestamps.txt file:

```JavaScript
FFTsamples.map(sample => sample.time * 1000).join('\n')
```

then running:

```shell
mp4fpsmod -o vfr.mp4 -t timecodes.txt ffmpeg-cfr.mp4 
```

Finally, merge the VFR video and your audio track into a CFR track with ffmpeg. The command below will select the video from the first input (the VFR video) and the audio from the second input (your MP3), and truncate the result to the shorter of the two in case of a slight discrepency (`-shortest`).

```shell
ffmpeg -i vfr.mp4 -i music.mp3 -map 0:v -map 1:a -shortest final-cfr.mp4
```