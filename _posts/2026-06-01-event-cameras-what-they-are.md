---
layout: post
title: "Event Cameras: What They Are and Why They're Weird"
date: 2026-06-01
category: research notes
tags: [neuromorphic, sensors, computer-vision, ORNL]
read_time: 6
---

Most cameras lie to you about time.

A standard frame-based camera — your phone, a webcam, a $50,000 cinema rig — works by telling every pixel to report its brightness all at once. The sensor says *now*, a million pixels blink simultaneously, and you get a frame. Repeat 30 times a second and you have video. This approach is so universal that it feels like a law of nature rather than a design decision.

It's a design decision. And for certain problems, it's the wrong one.

## The problem with frames

Frames waste information in two directions at once. Fast-moving objects blur across pixels between exposures — you've seen this as motion blur. Slow-moving or static objects get re-sampled identically across dozens of frames — you've seen this as video file sizes that seem absurd for how little actually changed.

The frame rate imposes a hard ceiling on what you can perceive. A standard camera running at 30 fps cannot detect a 1,000 Hz vibration. Full stop. The physics of sampling make it invisible. High-speed cameras push this ceiling — 1,000 fps, 10,000 fps — but at the cost of enormous data rates and power consumption, because you're now storing all those redundant static pixels at speed.

## A different question

What if instead of asking "what does the scene look like right now?" you asked "what *changed* since the last time I looked?"

That's the question a neuromorphic event camera answers. Instead of all pixels firing together on a clock, each pixel fires independently — and only when its local brightness changes by more than some threshold. The output isn't a frame. It's a stream of *events*, each one a tuple:

```
(x, y, timestamp, polarity)
```

Where `x` and `y` are the pixel coordinates, `timestamp` is when it fired (in microseconds), and `polarity` is whether brightness increased (+1) or decreased (−1).

Static pixels produce nothing. A moving edge produces a cascade of events tracing its path through the frame, timestamped at microsecond resolution.

## Why this is genuinely strange

The data structure breaks almost every assumption that computer vision is built on. There are no frames to index into. There's no concept of "frame rate." You can't feed raw events into a convolutional neural network expecting a 2D image. The temporal resolution is extraordinary — microseconds instead of milliseconds — but the spatial information arrives asynchronously, which means your algorithms have to think differently about time.

At ORNL this summer, I'm working with Prophesee event cameras as part of a multi-sensor vehicle tracking pipeline. The goal is to compare what event cameras can extract — geometric features, motion descriptors, velocity profiles — against what LiDAR and depth cameras see. The question isn't which sensor "wins." It's understanding what each one is actually good at, and where sensor fusion pays off.

## What they're actually good at

Event cameras shine when things move fast, lighting is dynamic, or power is constrained:

- **High-speed tracking** — a rotating wheel at 1,000 RPM is invisible to a 30 fps camera; an event camera resolves it cleanly
- **Low latency** — the first event fires within microseconds of a brightness change, not after a full frame exposure
- **Dynamic range** — roughly 120 dB vs. 60 dB for a standard camera, meaning they handle simultaneous bright sunlight and deep shadow without blowing out
- **Low power** — pixels that don't change consume almost nothing; power scales with scene activity

They're worse at anything requiring dense spatial information from static scenes, and the algorithms are immature compared to the decades of tooling built around frame-based computer vision.

## The interesting problem

Stereo event cameras — two event cameras separated by a baseline — can recover depth from disparity, exactly like human binocular vision. The challenge is that matching corresponding events between the two cameras is hard: you're matching sparse, asynchronous point streams rather than dense image patches. The grad student I'll be working alongside is tackling the stereo estimation piece; my focus will be on what you can do downstream once you have the tracks.

I'll write more about the specific processing pipeline — event accumulation, representations, the tracking stack — as the summer progresses. For now, the takeaway is this: event cameras aren't a better version of a normal camera. They're a fundamentally different sensor asking a fundamentally different question about the world. Getting good at them means unlearning some deeply baked assumptions about how vision data is supposed to look.

More soon.
