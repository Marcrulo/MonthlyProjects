---
title: 17. And Now I have a Master's Degree🎓
description: A quick review of my Master's thesis "Improving Hand Gesture Detection with Super-resolution Approaches"
published: true
image: 'thesis/pose_hands.png'
---

[Github repo](https://github.com/Marcrulo/masters-thesis) 

## [](#motivation)Motivation 💡
Hand gesture recognition (HGR) in hybrid meetings are yet to be explored, since the most advanced technology in this regard is to activate a thumbs up emote by holding up said gesture in front of a laptop camera. Bringing HGR into large meeting rooms requires a strategy for reliably recognizing gestures of especially small hands, which is what my thesis explores.

![motivation]({{ site.baseurl }}/assets/images/thesis/motivation.png "motivation")


## [](#outline)Outline 📋
The goal of the thesis is to explore and evaluate different methods for detecting hands and recognizing gestures, by mainly relying on open-source options, while also being mindful of feasability of inference time. The thesis mainly covers these following topics:
1. **Hand detection** (hand bounding boxes)
2. **Landmark localization** (hand joint keypoint)
3. **Gesture recognition** (single- and multiframe gesture classes)
4. **Edge Feasibility** (per-frame inference time on edge devices)

![themes]({{ site.baseurl }}/assets/images/thesis/themes.png "themes")


The goal is to identify meaningful improvements in HGR for the meeting room domain, while not focusing on absolute performance. This blog post will only touch briefly on the main findings and contributions, since fully covering a 85 page thesis, conducted over 6 months, will almost be as exhausting as watching that clip of [Skyler singing "Happy Birthday"](https://www.youtube.com/watch?v=n_GiCz7nPsU) again.

## [](#hands)Hands ✋

Using existing open-source hand detector models, small hands were barely detected. One reason could be that many hand detector models were trained to work on front-facing close-by hand detection, and not more troublesome real-life settings. There was not much to do about that, so in order to best utilize these models, tried various small object detection methods, that generally boil down to applying detection within crops of the image.

Techniques such as ***SAHI*** sweeps over the image and naively applies detection on all slices, whereas prior-based detection such as within a detected ***person and/or wrist crop*** reduces the number of detections, while also keeping the region-of-interest centered within the crop:

![hands-methods]({{ site.baseurl }}/assets/images/thesis/hands-methods.png "hands-methods")


The final method starts by detecting a person crop, followed by estimating body pose keypoints, and then super-resolving the padded crop. Hands detected within this crop are anchored to the respective wrist keypoints. This method relies on the full body as a prior, which works fabulously well - even on small hands. It does fail occasionally, especially if a body pose is harder to estimate:

![hands]({{ site.baseurl }}/assets/images/thesis/hands.png "hands")

Now that a hand crop is found, it's time to define the structure of the hand by estimaing hand joint keypoints/landmarks.

## [](#landmarks)Landmarks 📍

Another category of models, the "landmark localization" models, are applied within hand crops. **MediaPipe** is the industry standard, although I propose a new model, based on the SuperFAN architecture, which I have coined **Super*HAN***. The idea is to train a model that does both super-resolution and landmark localization jointly. MediaPipe is a great model in general, but SuperHAN provides solutions for low resolution hands, where MediaPipe struggles.

![landmarks]({{ site.baseurl }}/assets/images/thesis/landmarks.png "landmarks")

SuperHAN works great in these cases, but is in general outperformed by MediaPipe. Sticking to the MediaPipe landmarks, we move on to determining hand gestures.

## [](#gestures)Gestures 👋

Hand data come in 2 modalities; landmarks and images. Landmarks are much sparser and efficient than images, but are also more sensitive to noise. The gestures are therefore modelled in a multi-modal fashion, utilizing landmarks, hand crop, and the full image (as "context"). Due to the limitations of dynamic gesture datasets, sequences of single-frame gestures are therefore heuristically mapped to a dynamic gesture prediction.

The following two examples show predicted and ground-truth dynamic gesture sequences over time. This gives us an idea of how well it classifies gestures, as well as how predictions align temporally:

![dynamic-gestures-1]({{ site.baseurl }}/assets/images/thesis/dynamic-gestures-1.png "dynamic-gestures-1")

![dynamic-gestures-2]({{ site.baseurl }}/assets/images/thesis/dynamic-gestures-2.png "dynamic-gestures-2")

Admittedly, this static-to-dynamic heuristic did not perform particularly well in practice - these are some of the better examples. 

## [](#edge)Edge Devices 🔌

While the thesis doesn't optimize model for efficient edge devices performance, the models are still evaluated for edge devices to get an idea of how much to optimize. The goal is for the full edge *system* to keep performing at real-time at a frame-rate of 16 FPS (62.5 ms per frame). This means that the upper bound of inference time should not exceed 62.5ms. 

Two chipsets of interest are compared (denoted *Chip A* and *Chip B*), with chip B being much faster, and therefore of most fascinating. The following figure shows the accumulated inference time of the full pipeline, by considering the inference time of each step. This assumes 2 visible people, meaning that detection is done within 2 separate person crops.

![edge]({{ site.baseurl }}/assets/images/thesis/edge.png "edge")

Chip B does inference in the right order of magnitude of the upper bound for real-time performance. Many optimizations are possible, such as reducing the complexity of each component, or reducing the input size of the models, which obviously needs to balance accuracy and speed.


## [](#final)Final Thoughts 🎓🏁
There are so many other things worth discussing about this project, but I guess it's finally time to move on. It has been an interesting experience to deep dive into a problem for 6 months. Now, a day after defending my thesis (with a very pleasing grade) I'm still not able to fathom that my formal education is over. I have gone to school since I was 6 years old, and now I'm almost 25. I look forward to starting on my full time position at GN in August, and hope that I can bring some real value to the world, while staying true to myself. I'm so ready!😎
