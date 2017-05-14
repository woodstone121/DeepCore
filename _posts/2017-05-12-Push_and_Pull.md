---
title: Push and Pull
layout: post
author: Aleksey Vitebskiy
---

In the previous post, we've discussed the challenges of scaling object detection. We talked about how the traditional model of tiling images doesn't quite fit because we need to be able to detect objects that span multiple tiles. In this post, we'll examine the challenge of implementing a system that will do that efficiently. We will then introduce a different data processing method which aims to address the scalability issue while still being easy to extend.

# Traditional Batch Processing

In the traditional batch processing model, each step of an operation produces an intermediate result. That result is then fed to the next operation. The intermediate result is stored either on disk or in memory.

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

This is the oldest and the simplest way of composing different operations, this method dates back to mainframe days. Since each intermediate result is independent, we can add or change the operations in the chain easily. It's also simple to understand and debug. In fact, we don't even have to do it in the same program or script: we can run each step separately, passing the output data of each step into the next step.

There are some major disadvantages to this approach. Whether we store each intermediate result in memory or on disk, we're potentially creating full copies of the data in each step. In our example, we have to store three copies of each data. The process is also inherently serial, so we have to wait to download all the tiles before we can stitch them. Same issue with stitching the tiles then slicing them into sliding windows. Maybe we can improve on this.

# Combining Operations

Looking at the traditional batch processing chart, it's easy to see that there are simple ways to optimize that workflow. Let's examine how we would actually implement it using pseudocode with Python syntax, pseudo-Python if you will.

## Download and Stitch Tiles

The first part is tile stitching: there's no reason to store all the tiles before stitching them. Let's write a function that will download the tiles and stitch them together at the same time. For simplicity, we will assume that our image will always consist of full tiles. Our classifier will only return a single confidence value.

Here, we'll pretend that we have the following functions available:
* **create_blank_image(image_dimensions)** will create a blank image with the given dimensions in memory.
* **tiles_to_download(image_origin, image_dimensions)** will create a list of tiles that are needed to cover the requested area, which can just be a list of **(x, y)** tuples.
* **download_tile(tile_index)** will download a tile given a tile index.
* **calculate_image_rect(tile_index)** will take a tile index and convert it to a bounding box of that tile in the output image.


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

The next thing we can optimize is the sliding window slicer and the classifier. We don't have to generate every slice before we classify, we can just generate and classify one slice at a time.

Here we'll pretend that we have the following functions and properties available:
* **image.dimensions** there's a method in our **Image** class that returns image dimensions.
* **sliding_windows(image_dimensions, window_size, window_step)** generates rectangles using the sliding window algorithm.
* **classifier.classify(image)** classifies the given image and returns the confidence level.

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
        subset = image.subset(box)

        # classify the subset
        confidence = classifier.classify(image)

        # If the subset met our confidence threshold, add it to the list of predictions
        if confidence >= confidence_threshold:
            predictions.append((box, confidence))

    return predictions
{% endhighlight %}

# Better, but we Still didn't Solve our Problems

So far we've been able to eliminate having to save intermediate results of two steps. We no longer store all the tiles before combining them, and we no longer keep each subset before classifying it. This means that we've eliminated two out of three copies of data -- great! However, there are still some old issues remaining, and we actually added problems as well.

## Still Serial

Even though we've gained efficiency, we're still waiting on all tiles to download before starting the detection process. In fact, our download speed probably dropped, since we lost the ability to make multiple download requests at once. Let's rewrite the **downloadAndStitch** function to maximize the download speed.

Here, we'll pretend that we have the following function available in addition to what we defined in our original **downloadAndStitch** implementation:
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

    # Define a closure that will be called on each downloaded tile
    def on_tile(tile_coord, tile):
        # Figure out where the tile belongs
        image_rect = calculate_image_rect(tile_coord)

        # Copy the tile into the final image
        image.subset(image_rect) = tile

    # Now pass the closure function as an argument to the tile downloader
    download_tiles(tiles_to_download, on_tile)

    return image
{% endhighlight %}

This is getting complicated: we now have closures in play. It is actually one less line of code, but we're not showing the extra complexity of **download_tiles**. We can download and stitch at the same time now, so at least we can theoretically be as fast as the naive batch processing approach. Let's see what else can go wrong.

## Close Coupling Means Hard to Extend

Processing time on GPU compute instances is expensive, so we want to make it as efficient as possible. One idea is to ignore areas in which we know our object cannot appear. For example, if we're looking for ships, we shouldn't look on land; if we're looking for cars, we shouldn't look on water or around cliffs.

This means that we don't have to download as much, and we also don't have to classify as much. Classification is by far the most expensive part of our processing workflow, with tile download being the second slowest. We're going to go with the straightforward approach, so we'll ignore the asynchronous downloader and stitcher for now and go back to our original **downloadAndStitch** implementation.

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

This doesn't seem bad. We added one more argument to our function, rearranged some things, and added a couple of extra lines of code. However, we aren't finished yet. We still need to do the same for our **sliceAndClassify** function:

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

Now we had to modify a couple of functions, still not bad. What if we had to add another feature? We're now adding arguments to two different functions, we also still need to re-implement the asynchronous version of **downloadAndStitchWithRegionFilter**. What if the next operation is expensive as well? Maybe we'll need to think about chaining the asynchronous function calls.

## We are Still Not Scalable

By now we've completely forgotten about the problem we were trying to solve in the first place: we're still not scalable. This means that we still have a hard limit on how much area we can process at once, which means more processing nodes will be needed, which means more money spent.

Let's see if we can fix this problem. Maybe we'll have a tile cache that will allow us to dynamically stitch tiles as we need them. We'll probably want to download, stitch, and classify at the same time. We also want to make sure our region filter still works. This is starting to look like callback hell. What about thread synchronization? What if there are additional requirements in the future?

There has to be a better way.

# Introducing Pull Processing

## Push Processing

The approaches we examined so far have one thing in common: they finish a step of processing, then pass the result to the next step. All of operations are done sequentially, with the first operation in the chain following the second and so on. The code performs the operations in their logical order. We'll call this a "push" processing model. Doesn't code always follow the logical order of things?

## Pull Processing

It turns out that another approach is possible. Instead of sequencing operations, we instead organize them as a graph, with each node representing a single operation. The nodes are then connected in sequence, from source to sink. This doesn't sound too different, it's just a normal sequence of operations, right? The twist comes in how the processing is initiated.

Instead of the source node producing a result and passing it to the sink node, the sink node requests (or pulls) data from the source. The chain then propagates backwards, with each node requesting more data from its inputs to satisfy the request. Once the request chain propagated all the way back, the response comes back in the "forward" direction, with each successive node receiving the correct inputs to do its processing and pass the result down the chain. This is the "pull" processing model.

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

This is the sequence of events:

* Processing is initiated by the "Sliding Window" node, which asks the "Block Cache" for a subset.
* The "Block Cache" node then requests the tiles needed for that particular subset from the "Block Source" node.

At this point, the direction reverses. 

* The "Block Source" node downloads the requested tiles and passes them back to the "Block Cache" node.
* The "Block Cache" node mosaics the subset from the tiles it received and passes it to the "Sliding Window" node.
* The SlidingWindow node simply forwards the subset to the "Detector" node.
* The "Detector" and "Prediction Sink" nodes operate in a normal sequential fashion.

It's important to note that multiple requests and responses are in flight at once, the system is completely asynchronous. The illustration shows a sequence to satisfy a single request for clarity. The "Sliding Window" node doesn't just make one request, it actually makes ALL of the requests, all at once. Other nodes then process the data as it becomes available. Nodes operate independently, each in its own thread. Data is passed back and forth between nodes until all processing is done, that is, until there are no more requests or responses to process.

This system is very flexible, since we can mix and match the nodes in any way we want, as long as inputs and outputs match. Each node runs independently of others, which means maximum performance can be achieved at each step.

Following this abstraction model we can create a structured framework that will do all the housekeeping. The node implementers don't have to worry about threading or communication between nodes: they just write the normal sequential code to accomplish their task. By makind sure that only the framework code has to deal with threading issues, we make our software more reliable and maintanable.

## Pull Processing Programming Model

Pull processing does necessitate a different programming model. Instead of sequencing operations we define and connect the nodes, then just "push play" and wait on processing to complete. The above example would look like this in our pseudo-Python:


{% highlight python %}
# Create the nodes
source = BlockSource()
cache = BlockCache()
sliding_window = SlidingWindow()
detector = Detector()
sink = PredictionSink()

# Connect the nodes
source.output.connectWith(cache.input)
cache.output.connectWith(sliding_window.input)
sliding_window.output.connectWith(detector.input)
detector.output.connectWith(sink.input)

# Initiate processing, then wait for it to complete
sink.run()
sink.wait()

{% endhighlight %}

For simplicity, we didn't specify any initial parameters for the nodes. The parameters would normally be set before processing is initiated.

# DeepCore Processing Framework

The DeepCore processing framework implements the principals outlined in this post to achieve potentially unlimited scalability, while maintaining flexibility and performance. In the next post we will start examining each part of the framework so that we can learn how to use and extend it.

[traditional_batch_processing]: {{ site.baseurl }}/assets/images/2017-05-12-Push_and_Pull/traditional_batch_processing.png "Traditional Batch Processing"
{: width="100%"}


[pull_processing]: {{ site.baseurl }}/assets/images/2017-05-12-Push_and_Pull/pull_processing.gif "Pull Processing"
{: width="100%"}
