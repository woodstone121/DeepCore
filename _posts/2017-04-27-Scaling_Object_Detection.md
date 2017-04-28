---
title: Scaling Object Detection
layout: post
author: Aleksey Vitebskiy
---

Object detection in geospatial context presents some unique scalability challenges that are not normally tackled in the machine learning field. DeepCore attempts to address these challenges, though there's still work to be done. In this post we will discuss these issues and show how DeepCore will address them in the future.

# Why Geospatial Imagery is Special

In most applications, [CNNs][cnn] are used to classify a single image using a [CNNs][cnn] such as [LeNet][lenet], [AlexNet][alexnet], or [GoogLeNet][googlenet]. The classifier can then be used to localize objects within an image using the sliding window technique. Localizing object detection architectures such as [R-CNN][rcnn], [DetectNet][detectnet], or [YOLO][yolo] are able to localize objects without using a sliding window.

<table>
    <tr style="border: none; background-color: transparent;">
        <td markdown="span" align="center" style="padding: 0 .5em;">
            ![alt text][sliding_window]
        </td>
        <td markdown="span" align="center" style="padding: 0 0.5em;">
            ![alt text][detectnet_gif]
        </td>
    </tr>
    <tr style="border: none;">
        <td markdown="span" align="center" style="padding: 0;">
            Sliding window object detection
        </td>
        <td markdown="span" align="center"  style="padding: 0;">
            DetectNet object detection
        </td>
    </tr>
</table>

The problem is that most object detection workflows are geared towards detecting objects in photos or in video frames. Those images tend to be limited to a few megapixels. For those applications, a whole image can be processed at once. In fact, for video applications a localizing detector can be used with the input layer dimensions set to the video resolution, further maximizing detection performance.

Looking at the geospatial imagery applications, we see that imagery sometimes gets into gigapixel range. For example, the current theoretical limitation of OpenSpaceNet is about 2 gigapixels. However, that's only if the computer has enough memory to handle it. Image size doesn't fully describe memory requirements, since there are usually multiple image bands, and resampling can greatly increase memory requirements.

In practice, even if the machine has enough memory, we still have a lot of use cases where 2 gigapixels is not enough. 2 gigapixels at a 30 cm resolution is about 2,000 km<sup>2</sup>. As an example, metro Atlanta area is about 20,000 km<sup>2</sup>. We need to be able to scale much more than we currently can.

# Divide and Conquer

One of the ways [GIS][gis] and [Remote Sensing][remote_sensing] applications tackle the enourmous amounts of data they have to process is by dividing imagery into tiles. This allows to break the problem down into manageable pieces. Most web mapping services use 256&times;256 pixel image tiles, though tile sizes may vary. For example, Google Maps uses imagery of resolutions from 50 m to 15 cm, which cover the whole Earth. They store and serve this imagery in tiles sizes varying from 256&times;256 to 4096&times;4096. This lets them cover the majority of the Earth surface because all the tiles don't have to be loaded at the same time. They don't even have to be stored on the same computer.

<table>
    <tr style="border: none; background-color: transparent;">
        <td markdown="span" align="center" style="padding: 0 .5em;">
            ![alt text][stitched]
        </td>
        <td markdown="span" align="center" style="padding: 0 0.5em;">
            ![alt text][tiled]
        </td>
    </tr>
    <tr style="border: none;">
        <td markdown="span" align="center" style="padding: 0;">
            A satellite image of Atlanta Airport
        </td>
        <td markdown="span" align="center"  style="padding: 0;">
            Same image, but with tile borders shown
        </td>
    </tr>
</table>

This approach has other benefits as well. These include the ability to download and process multiple images at the same time. In fact, we can start processing tiles as soon as the first one loads without having to wait on the others to complete. Another benefit is that we can cache the tiles locally, so that we don't have to download them again if we need to look at the same area.

# The Edge Problem

The tiling strategy seems like the way to go for scaling [Remote Sensing][remote_sensing] applications, but it doesn't work for object detection. The problem is that if an object spans multiple tiles, the detector will never get a chance to look at a whole object, so the object will not be detected. This issue can be somewhat mitigated by dividing images into large subset, but the problem still remains. The sliding window has to run over the whole image in order to guarantee not to miss anything.

<table>
    <tr style="border: none; background-color: transparent;">
        <td markdown="span" align="center" style="padding: 0 0.5em;">
            ![alt text][split_object]
        </td>
    </tr>
    <tr style="border: none;">
        <td markdown="span" align="center" style="padding: 0;">
            Object split between two tiles
        </td>
    </tr>
</table>


# On-demand Tiles

The way to avoid loading the whole image at once is to load the tiles on-demand for each detector window. This creates very complicated issues, since we don't want to load the same tiles repeatedly. The tiles must be cached somehow, which further complicates the affair. The tiles have to be loaded in such a way to maximize throughput, while ensuring that the right tiles are always available, while avoiding removing the tiles that are still needed from the cache.

<table>
    <tr style="border: none; background-color: transparent;">
        <td markdown="span" align="center" style="padding: 0 0.5em;">
            ![alt text][combined_object]
        </td>
    </tr>
    <tr style="border: none;">
        <td markdown="span" align="center" style="padding: 0;">
            Object combined from two tiles
        </td>
    </tr>
</table>

Furthermore, it's beneficial to process the loaded subsets concurrently with loading more subsets. This normally leads to very complicated and error-prone code. Asynchronous code is notoriously difficult to debug and extend. Anybody experienced with [JavaScript][javascript] is familiar with "callback hell" that sometimes results from chaining asynchronous calls. Managing different threads and sharing data between them can turn into a real nightmare.

# The DeepCore Processing Framework

The DeepCore Processing Framework aims to solve these problems. This is the first in a series of articles in which we'll examine the design of the Processing Framework, which aims to make asynchronous components simple to implement and compose.

[cnn]: https://en.wikipedia.org/wiki/Convolutional_neural_network

[lenet]: http://yann.lecun.com/exdb/lenet/

[alexnet]: http://vision.stanford.edu/teaching/cs231b_spring1415/slides/alexnet_tugce_kyunghee.pdf

[googlenet]: https://research.google.com/pubs/pub43022.html

[rcnn]: https://blog.athelas.com/a-brief-history-of-cnns-in-image-segmentation-from-r-cnn-to-mask-r-cnn-34ea83205de4

[detectnet]: https://devblogs.nvidia.com/parallelforall/detectnet-deep-neural-network-object-detection-digits/

[yolo]: https://pjreddie.com/darknet/yolo/

[gis]: https://en.wikipedia.org/wiki/Geographic_information_system

[remote_sensing]: https://en.wikipedia.org/wiki/Remote_sensing

[javascript]: https://en.wikipedia.org/wiki/JavaScript

[sliding_window]: {{ site.baseurl }}/assets/images/2017-04-27-Scaling_Object_Detection/sliding_window.gif "Sliding Window"
{: width="384px"}

[detectnet_gif]: {{ site.baseurl }}/assets/images/2017-04-27-Scaling_Object_Detection/detectnet.gif "DetectNet"
{: width="384px"}

[stitched]: {{ site.baseurl }}/assets/images/2017-04-27-Scaling_Object_Detection/stitched_reduced.jpg "Satellite Image"
{: width="512px"}

[tiled]: {{ site.baseurl }}/assets/images/2017-04-27-Scaling_Object_Detection/tiled_reduced.jpg "Tiled Satellite Image"
{: width="512px"}

[split_object]: {{ site.baseurl }}/assets/images/2017-04-27-Scaling_Object_Detection/split_object.jpg "Tiled Satellite Image"

[combined_object]: {{ site.baseurl }}/assets/images/2017-04-27-Scaling_Object_Detection/combined_object.jpg "Tiled Satellite Image"