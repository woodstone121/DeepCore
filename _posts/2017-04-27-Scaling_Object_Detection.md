---
title: Scaling Object Detection
layout: post
author: Aleksey Vitebskiy
---

Object detection in geospatial context presents some unique scalability challenges that are not normally tackled in the machine learning field. DeepCore attempts to address these challenges, though there's still work to be done. In this post we will discuss these issues and show how DeepCore will address them in the future.

# Why Geospatial Imagery is Special

In most applications, CNNs are used to classify a single image using a CNN such as LeNet, AlexNet, or GoogLeNet. The classifier can then be used to localize objects within an image using the sliding window technique. Localizing object detection architectures such as R-CNN, DetectNet, and YOLO are able to localize objects without using a sliding window.

The problem is that most object detection workflows are geared towards detecting objects in photos or in video frames. Those images tend to be limited to a few megapixels. For those applications, a whole image can be processed at once. In fact, for video applications a localizing detector can be used with the input layer dimensions set to the video resolution, further maximizing detection performance.

Looking at the geospatial imagery applications, we see that imagery sometimes gets into gigapixel range. For example, the current theoretical limitation of OpenSpaceNet is about 2 gigapixels. However, that's only if the computer has enough memory to handle it. Image size doesn't fully describe memory requirements, since there are usually multiple image bands and resampling can greatly increase memory requirements.

In practice, even if the machine has enough memory, we still have a lot of use cases where 2 gigapixels is not enough. 2 gigapixels at a 30 cm resolution about 2,000 sq km. As an example, metro Atlanta area is about 20,000 sq km. We need to be able to scale much more than we currently can.

# Divide and Conquer

One of the ways GIS and Remote Sensing applications is by dividing imagery into tiles. This allows to break the problem down into managable pieces. Most web maping services use 256 x 256 pixel image tiles, though tile sizes may vary. For example, Google Maps uses imagery of resolutions from 50m to 15cm, which cover the whole Earth. They store and serve this imagery in tiles varying in sizes from 256x256 to 4096x4096. This lets them cover majority of the Earth surface because all the tiles don't have to be loaded at the same time, they don't even have to be stored on the same computer.

This approach has other benefits as well. These include the ability to download and process multiple images at the same time. In fact, we can start processing tiles as soon as the first one loads without having to wait on the others to complete. Another benefit is that we can cache the tiles locally so that we don't have to download them again if we need to look at the same area.

# The Edge Problem

The tiling strategy seems like the way to go for scaling remote sensing applications, but it doesn't work for object detection. The problem is that if an object spans multiple tiles, the detector will never get a chance to look at a whole object, so the object will not be detected. This issue can be somewhat mitigated by dividing images into large subset, but the problem still remains. The sliding window has to run over the whole image in order to guaranee not to miss anything.

# On-demand Tiles

The way to avoid loading the whole image at once is to load the tiles on-demand for each detector window. This creates very complicated issues, since we don't want to load the same tiles repeatedly. The tiles can be cached, but it's quite a complicated affair. The tiles have to be loaded in such a way to maximize throughput, while ensuring that the right tiles are always available, while avoiding removing the tiles that are still needed from the cache.

Furthemore, it's beneficial to process the loaded subsets concurrently with loading more subsets. This normally leads to very complicated and error-prone code. Asynchronous code is notoriously difficult to debug and extend. Anybody experienced with JavaScript is familiar with "callback hell" that sometimes results from chaining asynchronous calls.

# The DeepCore Processing Framework

The DeepCore Processing Framework aims to solve these problems. This is the first in a series of articles in which we'll examine the design of the Processing Framework, which aims to make asynchronous components simple to implement and compose.
