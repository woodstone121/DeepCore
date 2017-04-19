---
layout: post
title: Measuring Performance for Object Detectors - Part 1
author: Alan J. Schoen
---

This is the first part of a 2-part series of posts about measuring the accuracy of detector models.  As we develop more models, it's becoming important to have a standard score that tells us how well each model performed on a problem so we can choose the best one for each application.  In order to do this, we need to have ground truth images for reference.  These are created by having real live humans mark the images with the targets.  Then we compare the model output to the ground truth.  But there are important things we need to consider in order to define a scoring system that really shows us the best model for the job.

This post will cover the different decisions that went into the error calculation, and my next post will apply this to several different neural nets that me and my colleagues at DigitalGlobe have developed.

## Part 1: Defining a Metric

Consider this example: I've got a small image showing part of the famous airplane graveyard in Nevada.

![A section of the airplane graveyard][graveyard_plain]

My goal is to train a neural net that detects all of the airplanes.  We already know where the airplanes are because one of our interns helpfully marked up this image (that's the ground truth image).

![A section of the airplane graveyard, marked with polygons][graveyard_marked]

So now we can use the image and the ground truth markings to score models, but we'll need to define a good metric to measure model quality.  We can't score a model by just counting the number of airplanes that it found, because it's too easy to cheat.  A model could cover the whole image with a giant prediction, and that would find all the airplanes. 

![A bad prediction that catches all of the airplanes.][bad_prediction]

We found all the planes! Great, right?  Not really, because we just predicted that there were planes everywhere.  So if we want to score models well, we need to penalize models for making too many predictions.  To do this, we'll define a few quantities

* **True positive rate**: The number of positive examples that our model correctly found
* **False positive rate**: The number of times the model predicted an airplane, but there wasn't really a plane there
* **False negative rate**: When there's a plane, but the model doesn't find it

A good model will have a high **true positive rate**, and low **false positive** and **false negative** rates.  There are a few metrics used widely in machine learning which combine these numbers together into a single score.  Two common examples are [**ROC curves**](https://en.wikipedia.org/wiki/Receiver_operating_characteristic) and [**F1 scores**](https://en.wikipedia.org/wiki/F1_score).  We'd like to apply these scores to our models, but there's a problem.  These metrics are usually applied to data that's structured differently from our data, so there's some ambiguity in applying them to satellite imagery.  There's more than one way to define the **true positive rate** and the **false positive rate**.  We'll need to compare several different ways to do it and decide how to proceed.

Starting with the **true positive rate**, here are two ways to measure it.
1. Count each image pixel as a data point, and score the model based on how many pixels it classifies correctly.
2. Use an object-based approach, where an object is detected only if the prediction covers it sufficiently.  And we count false positives by finding predictions that missed the target and do not cover any objects sufficiently.

The first approach is really turning the problem into a [segmentation](https://leonardoaraujosantos.gitbooks.io/artificial-inteligence/content/image_segmentation.html) problem, which is a different thing from [object detection](https://www.mathworks.com/discovery/object-detection.html).  That's not ideal for this case, because it makes large objects more important than small objects.  If we're detecting aircraft, we want to weigh large aircraft like airliners equally with small aircraft like fighter jets.  So pixel-counting won't do.

The second approach makes more sense for us, but there are a few details to work out.  First, how do we decide whether or not an object is detected.

* How accurate does the detection box have to be before we call an object detected?
* Do we care if the boxes are oversized?
* Can one box detect more than one object?
* How should we count false positives?

#### Hitting the target
![Predictions at different offsets][offset_predictions]

Which of these images is a good detection?  We don't want to be too picky about the exact location of objects, since it's not really important in most applications, but we have to draw the line somewhere.  A simple way to deal with this is to set a threshold on the proportion of the object area covered by the prediction.  We can set a minimum of 0.4 to make sure we got a good chunk the object in the box.

#### Box Size
![Predictions at different sizes][sized_predictions]

What about boxes that contain an entire object, but are much larger than the actual object?  Is it even a valid detection if we have a ridiculously oversized box like the one we looked at at the beginning of the post? We want the boxes to represent the size of the object, and we'd like to avoid a scenario where a model gets rewarded for using big, imprecise boxes to increase its hit rate.  So let's introduce the [**Jaccard Index**](https://en.wikipedia.org/wiki/Jaccard_index), also known as [**intersection over union**](http://www.pyimagesearch.com/2016/11/07/intersection-over-union-iou-for-object-detection/) (**IoU**).

Before computing the **Jaccard index**, we'll switch from using the precise object markings to using a box circumscribing the object because this gives better results.  

![Predictions at different sizes][sized_predictions_jacc]  
From looking at these images, it looks like a **Jaccard index** over 0.45 is a good detection.  A number below that is a sign that the box is too big or too small.  We can use a Jaccard cutoff instead of a proportion cutoff.


#### Multiple Detections


![Two targets, one match][2pred2] &nbsp;&nbsp; <font size="+3"> <b> or </b> </font> &nbsp;&nbsp;
![Two targets, two matches][2pred3] 


Should we let a single box detect multiple airplanes?  If you just want to find airplanes, you might not care whether a box touches one plane or more than one.  But if your goal is to count planes, then you really want to enforce strict correspondence between boxes and planes.  If you want to restrict boxes to planes, then you can enforce that rule in the error calculation. My error calculator supports both.

#### False Positives
![This prediction missed the target completely.][total_miss]
![Here we have two predictions that hit the target.][double_down] 

The picture on the left is clearly a miss because the box just plain missed the target.  But what about the picture on the right?  Should we count both of those as hits, or count just one?  Or should we discount the overlapping portion in some way?  This depends on what we're using the model for.  If your ultimate goal is to count planes, then you should penalize the model for putting more than one detection box over the same plane.  If you just want to find planes at any cost, then you could allow multiple detections of the same plane, or merge all the overlapping boxes into one and allow multiple detections.  It really depends on what you're planning to use the model for.

## Conclusions

Deciding how to score true positives, false positives, and false negatives will determine which models look better than others, so it's important to make decisions that make sense for your particular application.  I made my own decisions on these questions, based on what I thought our model would be used for.  Then I wrote an accuracy calculator to score models against test imagery based on my criteria.  In some cases, I supported multiple options because I thought we might need to use a different measurement for different problems.

In my next post, I will compare models against each other using my accuracy calculator.

![alt-text]({{ site.baseurl }}/images/banner2.png)

[graveyard_plain]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/graveyard_plain.png "A section of the airplane graveyard"
{: width="950px"}

[graveyard_marked]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/graveyard_marked.png "A section of the airplane graveyard, marked with polygons"
{: width="950px"}

[bad_prediction]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/bad_prediction.png "A bad prediction that catches all of the airplanes."
{: width="950px"}

[offset_predictions]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/offset_predictions_prop.png "Predictions at different offsets"
{: width="1000px"}

[sized_predictions]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/sized_predictions.png "Predictions at different sizes"
{: width="1000px"}

[sized_predictions_jacc]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/sized_predictions_jacc.png "Predictions at different sizes"
{: width="1000px"}


[2pred2]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/2pred2.png "Two targets, one match"
{: width="250px"}

[2pred3]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/2pred3.png "Two targets, two matches"
{: width="250px"}

[total_miss]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/total_miss.png "This prediction missed the target completely."
{: width="950px"}

[double_down]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/double_down.png "Here we have two predictions that hit the target."
{: width="950px"}
