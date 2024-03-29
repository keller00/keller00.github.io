---
title: FFMPEG Combining and Separating audio streams
date: 2016-11-27T00:00:00-04:00
subtitle: Examples and Explanations
tags:
- ffmpeg
- audio stream
- audio streams
- combine
- separate
---
FFMPEG is an open source, wonderful tool to transcode and manipulate picture/audio/video files.  
It can do so many different things, it's unbelievable. Therefore its syntax can be overwhelming.  

So I'll provide 2 examples and explain them in detail while having 2 specific goals in mind:  
1. Combine X audio streams into a single stream with the channels coming from those streams' channels in order  
2. Separate a single audio stream's channels into different audio streams  

As an example consider this file (I'm only showing part of ffprobe's output to save space):  

```
Stream #0:0(eng): Video: prores (apch / 0x68637061), yuv422p10le(bt709), 1920x1080, 220043 kb/s, SAR 1:1 DAR 16:9, 29.97

Stream #0:1(eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, mono, s32 (24 bit), 1152 kb/s (default)
Stream #0:2(eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, mono, s32 (24 bit), 1152 kb/s (default)
Stream #0:3(eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, mono, s32 (24 bit), 1152 kb/s (default)
Stream #0:4(eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, mono, s32 (24 bit), 1152 kb/s (default)
Stream #0:5(eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, mono, s32 (24 bit), 1152 kb/s (default)
Stream #0:6(eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, mono, s32 (24 bit), 1152 kb/s (default)
Stream #0:7(eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, mono, s32 (24 bit), 1152 kb/s (default)
Stream #0:8(eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, mono, s32 (24 bit), 1152 kb/s (default)

Stream #0:9(eng): Data: none (tmcd / 0x64636D74)
```

#### So how could you turn these 8 mono streams into a single 8 channel stream?  

```
ffmpeg -i input.mov -filter_complex "[0:a]amerge=inputs=8[a]" -map 0:v -map "[a]"  -map 0:d -c:v copy -c:d copy -c:a pcm_s24le output.mov -y
```

What does all that exactly do? Glad you asked!

```
-i input.mov --- specifies an input-file, more -i will add additional input files, numbered from 0
-filter_complex --- tells ffmpeg that we're going to use a complex filter, such as amerge, which does the combining for us
"[0:a]amerge=inputs=8[a]" --- [0:a] means that we're using the first input file's audio streams, the we specify the number of audio channels in the output file with inputs=8, then we finally name the output of this filter "a" with [a]
-map 0:v -map "[a]"  -map 0:d --- tells ffmpeg how to order the streams in the output file, first input file's video streams, then "a" from amerge and finally the data streams of the first input file
-c:v copy -c:d copy -c:a pcm_s24le --- tells ffmpeg to simply copy video and data streams and to encode audio streams using pcm_s24le encoder
output.mov -y --- specifies output file and -y tells ffmpeg that it's okay to overwrite the file, if it already exists
```

#### Now how could we turn this back into what it was? (1 stream, 8 channels to 8 streams 1 channel)?  

```
ffmpeg -i input.mov -filter_complex "[0:a]pan=mono|c0=c0[a0]; [0:a]pan=mono|c0=c1[a1]; [0:a]pan=mono|c0=c2[a2]; [0:a]pan=mono|c0=c3[a3]; [0:a]pan=mono|c0=c4[a4]; [0:a]pan=mono|c0=c5[a5]; [0:a]pan=mono|c0=c6[a6 [0:a]pan=mono|c0=c7[a7]" -map 0:v -map "[a0]" -map "[a1]" -map "[a2]" -map "[a3]" -map "[a4]" -map "[a5]" -map "[a6]" -map "[a7]" -map 0:d -c copy -c:a pcm_s24le output.mov -y
```

Parts similar to the previous example are left out to shorten this post

```
-filter_complex --- so we declare complex filters again, but we'll use the pan filter this time around
"[0:a]pan=mono|c0=c0[a0]" --- tells ffmpeg to use the pan filter and create a mono stream using the audio streams of the first input file, then we name it "a0" notice how multiple options can be added to the complex filter by adding a  ';' and then the next option at to the string, repeat as many times as many streams are desired (8 in this example)
-map 0:v -map "[a0]" -map "[a1]" ... -map a:d --- maps first the video, then the seperated audio streams and finally the audio streams
```
