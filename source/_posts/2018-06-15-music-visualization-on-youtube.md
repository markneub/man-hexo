---
title: Music Visualization on Youtube
date: 2018-06-15 15:09:43
tags:
---

<div class="aspect-ratio sixteen-nine"><iframe width="640" height="390" src="//www.youtube.com/embed/TyhisYKU2Dg" frameborder="0" allowfullscreen="" style="margin-bottom: 20px;"></iframe></div>

Recently, I came across the [butterchurn](https://github.com/jberg/butterchurn) library, which brings the classic [Milkdrop](https://en.wikipedia.org/wiki/MilkDrop) music visualizer to the web via WebGL. I was also talking to a friend about his recently released album, and had a fun project idea — what if I could prerender some of my favorite Milkdrop presets to the music and upload it to YouTube?

Butterchurn is designed to render frames to a <canvas> HTML element, but I wanted to get the output into a video file on disk that I'd be able to upload to YouTube. Some quick research turned up [CCapture.js](), a library for capturing <canvas> contents and allowing for export to a variety of formats. However, I quickly discovered that exporting each frame at 60fps in real time wasn't going to work; the captures just couldn't keep up, even at low resolution. Since the visuals were being prerendered, this was also a great opportunity to crank up the quality. My next move was to make some modifications to butterchurn to separate the "listening" phase from the "rendering" phase. In this case, listening refers to sampling the audio at around animation speed (60fps) and generating a sample containing, among other things, the fast fourier transform of the audio signal at the time of the sample. Freed up from the constraints of having to render and write out each frame, the listening phase could easily proceed at real time, with the expensive rendering taking as long as necessary in a second pass.

For the rendering phase, I could now iterate over the collection of samples I took in the listening phase, use butterchurn to render each frame, and then have CCapture.js send the PNG data over a websocket to a Nodejs server, which would save the frames to disk and ultimately use ffmpeg to stitch all the frames together into a video. Great, except the resulting video didn't quite match up with the music audio! After puzzling about this a little bit, I realized that the butterchurn's sample rate was not an consistent 60 times per second, and varied pretty widely. Fortunately, each sample I was grabbing (essentially dumping the state of the renderer into each sample) also included a timestamp indicating how far into the track it was captured. Armed with this information, I found [mp4fpsmod](https://github.com/nu774/mp4fpsmod), which would allow me to modify my 60 fps CFR (constant frame rate) video into a VFR (variable frame rate) video, with each frame's position indicated by the timestamp of the sample I used to render it.

The final step I took was to use ffmpeg to process the video one more time, to add the audio track and to convert it back into CFR, approximating the VFR clip I created in the previous result. (This happens by dropping and duplicating frames as appropriate.) The easiest part of the whole process, of course, was just dragging and dropping onto YouTube :).

If you'd like to run the code yourself, the code for this project includes a modified version of butterchurn that splits the listening and rendering phases, and my glue code that adds in CCapture.js. Beware, my goal was to hack my way to a working solution as quickly as possible so it's quite ugly.

- milkdrop-prerenderer
- butterchurn

You'll also need mp4fpsmod, which you'll need to compile and install yourself.

mp4fps mod command

ffmpeg command
