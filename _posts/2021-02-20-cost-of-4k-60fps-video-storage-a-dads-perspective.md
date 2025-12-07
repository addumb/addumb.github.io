---
layout: post
title: The Cost of 4K 60fps video storage, a dad's perspective
date: 2021-02-20 00:00:00.000000000 -07:00
type: post
published: true
status: publish
description: Recording 4k 60fps iPhone videos costs nearly nothing for an individual
keywords: 
categories:
- Tech
tags:
- Tech
---
I'm a dad, now! Yay! My son is awesome. Being a dad during the COVID-19 pandemic is awful in myriad ways. It has silver linings, but this post isn't about that.

# I ran out of iCloud storage
I've been taking 4K 60fps videos of my son doing things which I find earth-shattering: looking at me, sneezing, babbling, laughing, eating, even bathing and peeing LOL. I'm taking these in 4k 60fps on my iPhone whatever (11 pro, midnight green for sure) which offers a simple set of toggles to do so. Obviously, recording video at 4x the resoultion and 2x the framerate will likely cost in the ballpark of 8x in storage according to napkin math. This is the same ballpark as an order of magnitude increase in storage for video. It's not, but my napkin math instinct should have informed me.

These toggles are, to my new dad perspective, future proofing my memories. Back in the days before the pandemic, people would have high school graduation parties. At my own, my parents lovingly trawled through the family archives to unearth the most dubiously endearing but truthfully embarrassing photos and videos they could find. Great success on there part, if memory serves me well.

I want a trove of ancient videos to share with the world when my son hits these comming of age milestones. I want to show him how I'm such an idiot that I sacrificed future flexibility on my part, fitting-in better on his part, and generally not being embarrassed on all parts. I'm already so proud of him, I cannot imagine how proud I'll be if he has one of these shindigs to celebrate a major achievement among his peers. Him taking a crap is a major achievement in my book, so I'm just gonna be constantly gushing about how awesome he is. I have to keep that discount for myself.

So, I ran out of iCloud storage, of course. Who tf doesn't? That's the entire point of iCloud storage: to run out and pay apple for more "backups." This got me thinking, how am I gonna save these 4k 60fps videos? How am I going to archive them and then retrieve them at a later point? Well, let's start with Apple's estimates of storage costs of these videos. The iPhone settings app has estimates listed under the camera details:

> A minute of video will be approximately:
> * 40 MB with 720p HD [editorial: LOL @ HD] at 30 fps (space saver)
> * 60 MB with 1080p HD at 30 fps (default)
> * 90 MB with 1080p HD at 60 fps (smoother)
> * 135 MB with 4k [editorial: why no "UHD"?] at 24 fps (film style) 🙄
> * 170 MB with 4k at 30 fps (higher resolution)
> * 400 MB with 4k at 60 fps (higher resolution, smaller)

1080p @30fps is 60MB/minute or 1MB/s. 4k @60fps is 400MB per minute, or 6.66667whatever MB/s. Let's invert it so we don't have repeating decimals, they're annoying in text: 1080p @30fps is 0.15 times the size of 4k @60fps. Cool, 8x (or 1/0.125x) was pretty close! Let's just blame the difference as a rounding compromise at Apple to keep the camera settings nice and neat, though imprecise. We're still gonna use their numbers for now, though. My napkin math is supported by only one scientist.

# How much video do I take?
Well this is highly variable, but if there's one thing I learned to get my Physics degree, it's that you can approximate the shape of a cow as roughly a sphere. So let's approximate. Let's see how much I've taken at 4k 60fps so far:

```
$ ls -l /mnt/lilstorage/Videos/Other/iPhone/2021-02-20/|wc -l
310
```
Ok, 310 videos. Let's see how to get their duration. (googling...) looks like `mediainfo` will do it, so `sudo apt-get install mediainfo` or whatever, then away we go:
```
$ for f in *; do mediainfo $f | grep ^Duration | head -1; done
... snip
Duration                                 : 1 min 4 s
Duration                                 : 50 s 277 ms
... snip
```
Great, the tool unhelpfully assumes that I, a human, am not a computer. Ends up humans can compute, so that's annoying. Oh, but it supports * args and has a JSON output option:
```
$ mediainfo --Output=JSON * | jq .
... snip uhhhh on second thought, you don't want to see this
```
Ok, so it has some structure. For each file, it outputs 3 "tracks": general, video, and audio. But it already looks a bit askance. I need to compare a 30fps and 60fps file, a 4k and a 1080p file, and maybe even the full cross to understand this tool. One 4k 60fps file I have is named IMG_4165.MOV, and a 1080p 30fps file is IMG_3603.MOV. Let's compare their outputs:
```
$ mediainfo --Output=JSON IMG_4165.MOV IMG_3603.MOV | jq .
... snip 
FrameRate
Duration
Height
Width
... snip
```
Yeah, I crapped out, but that's the point at least: it's getting a bit more complicated just to tally up the duration of 4k vs 1080p videos I've taken. I'll plop this out to a file and reat it into ipython:
```
$ mediainfo --Output=JSON * | jq '.[].media.track[1] | "\(.Width) \(.Height) \(.FrameRate) \(.Duration)" | sed 's/"//' > ~/derp.txt
```
`jq` cares about quotes, I don't.

Then let's load it up and group things together:
```python
#!/usr/bin/env python3
derp = open("derp.txt", "r").readlines()

from collections import namedtuple, defaultdict

vidfile = namedtuple("vidfile", "width height framerate duration")
vidfiles = [vidfile(*l.strip().split(" ")) for l in derp]


def framebucket(fps):
    fps = float(fps)
    if fps < 20:
        return "timelapse"
    elif fps < 26:
        return "24"
    elif fps < 35:
        return "30"
    elif fps < 65:
        return "60"
    else:
        return "slowmo"


vid = namedtuple("vid", "res framebucket duration")
vids = [
    vid(int(v.width) * int(v.height), framebucket(v.framerate), v.duration)
    for v in vidfiles
]

summary = defaultdict(int)
for v in vids:
    summary[f"{v.res} @ {v.framebucket}"] += float(v.duration)

for k, v in sorted(summary.items()):
    print(k, v)

```
This outputs that I make 5x more 1080p @ 30fps videos than 4k @ 60fps videos:
```
1701120 @ 24 15.255
1701120 @ 60 42.568999999999996
2073600 @ 24 896.58
2073600 @ 30 10020.490000000009
2073600 @ 60 313.47099999999995
2073600 @ slowmo 18.997
2764800 @ 30 1.8079999999999998
8294400 @ 30 233.902
8294400 @ 60 2065.2369999999996
921600 @ 30 632.647
```
Ignore the rounding errors, you'll see 10020.5 and 2065.237 for each, respectively.

# How much video *will* I take?
I confess: I take a bunch of very short videos of specific interactions. I'm trying to take longer videos, but it's hard to do that while the subject tries their best to get into trouble.

Well my son is 9 months old (minus 3 days) and I've taken lots. My phone broke and I had to swap-up when he was only 1 month old and that's what these files cover. So 10020 seconds in 8 months of 1080p 30fps video and 2065 seconds of 4k 60fps video, or on rough average: 0.347917 hours per month of 1080p 30fps and only 0.07170138 hours per month of 4k 60fps.

If we pretend I keep my habit as-is, by the time he's 18, I'll have 75 hours of ancient 1080p 30fps video and a measly 15.5 hours of less-ancient 4k 60fps video. Decent to choose from for a simple carousel style loop.

*Phew*! I was worried I'd need to factor in how 1080p -> 4k -> 8k etc progression will happen and estimate filesize growth over time.

By Apple's estimates, my 15.5 hours of 4k 60fps video will take only about 360GB to store. Easy peasy!

# Technology progression costs more storage!
But wait, I do need to estimate the progression! I don't have the energy to estimate, so I'll over-estimate by supposing my entire archive is best approximated as being 400MB per minute (nullifying the fun I just had): 2TB. Big whoop. I guess I won't get that NAS and will just shove them into S3 glacier for now.
