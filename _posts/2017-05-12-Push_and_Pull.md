---
title: Push and Pull
layout: post
author: Aleksey Vitebskiy
---

In the previous post we've discussed the challenges of scaling object detection. We talked about how the traditional model of tiling images doesn't quite fit becase we need to be able to detect objects that span multiple tiles. In this post we'll examine the challenge of implementing a system that will do that efficiently. We will then introduce a different data processing method which aims to address the scalability issue while still being easy to extend.

# Traditional Batch Processing

In the traditional batch processing model, each step of an operation produces an intermediate result. That result is then fed to the next operation. The intermediate result is either stored on disk, or in memory.


<table>
    <tr style="border: none; background-color: transparent;">
        <td markdown="span" align="center" style="padding: 0 0.5em;">
            ![alt text][traditional_batch_processing]
        </td>
    </tr>
    <tr style="border: none;">
        <td markdown="span" align="center" style="padding: 0;">
            Traditional batch processing
        </td>
    </tr>
</table>

This is the oldest and the simplest way of composing different operations, that dates back to mainframe days. Since each intermediate result is independent, we can add or change the operations in the chain easily. It's also simple to understand and debug. In fact, we don't even have to do it in the same program or script: we can run each step separately, passing the output data into the next step. 

There are some major disadvantages to this approach. Whether we store each intermediate result in memory or on disk, we're essentially duplicating all the data in first three steps. The process is also inherently serial, so we have to wait to download all the tiles before we can stitch them. Same issue with stitching the tiles, then slicing them into sliding windows. Maybe we can improve on this.

# Combining Operations

Now looking at the traditional batch processing chart, it's easy to see that there are simple ways to optimize that workflow. Lets examine how we would actually implement using pseudocode with Python syntax, pseudo-Python if you will.

## Download and Stitch Tiles

The first part is tile stitching: there's no reason to store all the tiles before stitching them. Let's write a function that will download the tiles and stitch them together at the same time. For simplicity our image will always consist of full tiles. Our classifier will only return a single confidence value.

Here we'll pretend that we have the following functions available:
* **create_blank_image(image_dimensions)** will create a blank image in memory with the dimensions given
* **tiles_to_download(image_origin, image_dimensions)** will create a list of tile indexes, which can just be a list of **(x, y)** tuples.
* **download_tile(tile_index)** will download a tile given a tile index
* **calculate_image_rect(tile_index)** will take a tile index and convert it to a bounding box of that tile in the output image


This is the pseudocode for our **downloadAndStitch** function:

{% highlight python %}
#
# Downloads the tiles and stitches image together. Returns the complete image.
#
def downloadAndStitch(image_origin, image_dimensions):
    # Create a blank image that we will save our tiles into
    image = create_blank_image(image_dimensions)

    # Calculate which tiles we need to download
    tiles_to_download = get_tile_indexes(image_origin, image_dimensions)

    for tile_index in tiles_to_download:
        # Download the tile image
        tile = download_tile(tile_index)

        # Figure out where the tile belongs
        image_rect = calculate_image_rect(tile_index)

        # Copy the tile into the final image
        image.subset(image_rect) = tile

    return image
{% endhighlight %}

## Classify Each Subset Using the Sliding Window Algorithm

The next thing we can optimize is the sliding window slicer and classifier. We don't have to generate
every slice before we classify, we can just generate and classify one slice at a time.

Here we'll pretend that we have the following functions and properties available:
* **image.dimensions** there's a method in our **Image** class that returns image dimensions
* **sliding_windows(image_dimensions, window_size, window_step)** generates rectangles using the sliding window algorithm
* **classifier.classify(image)** classifies the given image and returns the confidence level

{% highlight python %}
#
# Takes slices of an image using the sliding window model and classifies each one,
# outputs predictions.
# 
# Each prediction will include the prediction bounding box and confidence.
#
def sliceAndClassify(image, window_size, window_step, classifier, confidence_threshold):
    # generate a bounding box for each window
    boxes = sliding_windows(image.dimensions, window_size, window_step)

    # now take an image subset for each bounding box and classify it
    predictions = []
    for box in boxes:
        # get the subset from the image
        subset = image(box)

        # classify the subset
        confidence = classifier.classify(image)

        # If the subset met our confidence threshold, add it to the list of predictions
        if confidence >= confidence_threshold:
            predictions.append((box, confidence))

    return predictions
{% endhighlight %}

# Better, but we Still didn't Solve our Problems

So far we've been able to eliminate having to save intermediate results of two steps. We no longer store all the tiles before combining them, and we no longer keep each subset before classifying it. This means that we've eliminated two out of three copies of data, great! Well, it's better, but there are still some old issues remaning, and we actually added problems as well.

## Still Serial

Even though we've gained efficiency, we're still waiting on all tiles to download before starting the detection process. In fact, our download speed probably dropped, since we lost the ability to make multiple download requests at once. Well, at least fix our downloads: let's rewrite our **downloadAndStitch** function.

Here we'll pretend that we have the following function available in addition to what we defined in our original **downloadAndStitch** implementation:
* **download_tiles(tiles_to_download, on_tile)** downloads the requested tiles as quickly as possible, calls the given **on_tile(tile_coord, tile)** function as soon as a tile is downloaded

{% highlight python %}
#
# Downloads the tiles and stitches image together
#
def downloadAndStitchAsync(image_origin, image_dimensions):
    # Create a blank image that we will save our tiles into
    image = create_blank_image(image_dimensions)

    # Calculate which tiles we need to download
    tiles_to_download = get_tile_indexes(image_origin, image_dimensions)

    # define a closure that will be called on each downloaded tile
    def on_tile(tile_coord, tile):
        # Figure out where the tile belongs
        image_rect = calculate_image_rect(tile_coord)

        # Copy the tile into the final image
        image.subset(image_rect) = tile

    # now pass the closure function as an argument to the tile downloader
    download_tiles(tiles_to_download, on_tile)

    return image
{% endhighlight %}

OK, this is getting complicated. We now have closures in play. It is actually one less line of code, but we're not showing the extra complexity of **download_tiles**. We can download and stitch at the same time now, so that's good. At least we can theorectically be as fast as the naive batch processing approach. Let's see what else can go wrong.

## Close Coupling Means Hard to Extend

So we have our application working in production now, it's working great. However, processing time on GPU compute instances is expensive. We want to make it more efficient. Somebody has an idea: how about we ignore areas in which we know our object cannot appear? For example, if we're looking for ships, we shouldn't look on land; if we're looking for cars, we shouldn't look on water or around cliffs. 

This means that we won't have to download as much, we also won't have to classify as much. Classification is by far the most expensive part of our processing workflow, with tile download being the second slowest. We're going to go with the straighforward approach, so we'll ignore the asynchronous downloader and stitcher for now and go back to our original **downloadAndStitch** implementation.

Here we'll pretend that we have the following class available in addition to what we defined in our **downloadAndStitch** implementation:

* **class RegionFilter** that has a method **contains(rect)**. The method returns **True** if we should include the rectangle in question, and **False** if we shouldn't.

{% highlight python %}
#
# Downloads the tiles and stitches image together, optionally allows omitting tiles.
#
def downloadAndStitchWithRegionFilter(image_origin, image_dimensions, region_filter=None):
    # Create a blank image that we will save our tiles into
    image = create_blank_image(image_dimensions)

    # Calculate which tiles we need to download
    tiles_to_download = get_tile_indexes(image_origin, image_dimensions)

    for tile_coord in tiles_to_download:
        # Figure out where the tile belongs
        image_rect = calculate_image_rect(tile_coord)

        # If we don't need to look at the tile, skip downloading it
        if region_filter is not None and not region_filter.contains(image_rect):
            continue

        # Download the tile image
        tile = download_tile(tile_coord)

        # Copy the tile into the final image
        image.subset(image_rect) = tile

    return image
{% endhighlight %}

Well, this wasn't so bad. We added one more argument to our function, rearranged some things, and added a couple of extra lines of code. Wait, we aren't finished yet. We still need to do the same for our **sliceAndClassify** function:

{% highlight python %}
#
# Takes slices of an image using the sliding window model and classifies each one,
# outputs predictions, optionally allows omitting areas of the image.
# 
# Each prediction will include the prediction bounding box and confidence.
#
def sliceAndClassifyWithRegionFilter(image, window_size, window_step, classifier, confidence_threshold, region_filter=None):
    # Generate a bounding box for each window
    boxes = sliding_windows(image.dimensions, window_size, window_step)

    # Now take an image subset for each bounding box and classify it
    predictions = []
    for box in boxes:
        # If we don't need to look at this bounding box, skip it
        if region_filter is not None and not region_filter.contains(image_rect):
            continue

        # Get the subset from the image
        subset = image(box)

        # Classify the subset
        confidence = classifier.classify(image)

        # If the subset met our confidence threshold, add it to the list of predictions
        if confidence >= confidence_threshold:
            predictions.append((box, confidence))

    return predictions
{% endhighlight %}

OK, so we had to modify a couple of functions now, still not bad. What if we had to add another feature? We're now adding arguments to two different functions, we also still need to re-implement the asynchronous version of **downloadAndStitchWithRegionFilter**. What if the next operation is expensive as well? Maybe we'll need to think about chaining the asynchronous function calls.

## We are Still Not Scalable

By now we've completely forgotten about the problem we were trying to solve in the first place: we're still not scalable. This means that we still have a hard limit on how much area we can process at once, which means more processing nodes will be needed, which means more money spent.

Let's see if we can fix this problem. Maybe we'll have a tile cache that will allow us to dynamically stitch tiles as we need them. We'll probably want to download, stitch, and classify at the same time. We also want to make sure our region filter still works. This is starting to look like callback hell. What about thread synchronization? What if there are additional requirements in the future? 

There has to be a better way.

# Introducing Pull Processing

The approaches we examined so far have one thing in common: they finish a step of processing, then pass the result to the next step. What if the result (sink) initiated processing instead. Instead of going from source to sink, we'll go the opposite way. Instead of sliding window being told to slice up an image, what if the sliding window asked which part of image it wants to be retrieved? The data is always retrieved on demand, each step of the process is either requesting data, or processing the response it received and forwarding it to the next step.

Let's take a look at an example:

<table>
    <tr style="border: none; background-color: transparent;">
        <td markdown="span" align="center" style="padding: 0 0.5em;">
            ![alt text][pull_processing]
        </td>
    </tr>
    <tr style="border: none;">
        <td markdown="span" align="center" style="padding: 0;">
            Pull processing
        </td>
    </tr>
</table>


[traditional_batch_processing]: {{ site.baseurl }}/assets/images/2017-05-12-Push_and_Pull/traditional_batch_processing.png "Traditional Batch Processing"
{: width="768px"}


[pull_processing]: {{ site.baseurl }}/assets/images/2017-05-12-Push_and_Pull/pull_processing.gif "Pull Processing"
{: width="768px"}
