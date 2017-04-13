---
layout: post
title: Measuring Performance for Object Detectors - Part 1
author: Alan J. Schoen
---

# Measuring Performance for Object Detectors

## Part 1: Defining a Metric

We build a lot of neural networks to detect objects, using different techniques, different machine learning packages, and different training sets.  It's important not to lose sight of the big picture: finding the best model to detect the things we're looking for.  To do this, we need a measurement of accuracy that lets us reduce everything to a single number.  In part 1 of this series of blog posts, I'll outline some of the challenges in defining a good metric to measure the quality of a model.

For the last few months, I have been training neural networks to detect different kinds of aircraft.  I create training sets using DigitalGlobe imagery and a python library that I am writing to turn this imagery into tiles I can feed into neural networks.  Now that we're training a lot of neural networks, we need a good way to decide which are the best ones.

Consider this example.  I've got a small image showing part of the famous airplane graveyard in Nevada.

![alt text][graveyard_plain]

I have an image of the airplane graveyard in Nevada, and I want to detect all of the airplanes.  We already know where the airplanes are because one of our interns helpfully marked up this image.

![alt text][graveyard_marked]

So now we can use the image and the ground truth markings to score models, but we'll need to define a good metric to measure model quality.  We can't score a model by just counting the number of airplanes that it found, becuase its too easy to cheat.  A model could cover the whole image with a giant prediction, and that would find all the airplanes. 

![alt text][bad_prediction]

We found all the planes! Great, right?  Not really, because we just predicted that there were planes everywhere.  So if we want to score models well, we need to penalize models for making too many predictions.  To do this, we'll define a few quantities

* **True positive rate**: The number of positive examples that our model correctly found
* **False positive rate**: The number of times the model predicted an airplane, but there wasn't really a plane there
* **False negative rate**: When there's a plane, but the model doesn't find it

A good model will have a high **true positive rate**, and low **false positive** and **false negative** rates.  There are a few metrics used widely in machine learning which combine these numbers together into a single score.  Two common examples are **ROC curves** and **F1** scores.  We'd like to apply these scores to our models, but there's a problem.  Our data doesn't look like normal data, so there's more than one way to define the **true positive rate** and the **false positive rate**.  We'll need to compare several different ways to do it and decide how to proceed.

Starting with the **true positive rate**, here are two ways to measure it.
1. Count each image pixel as a data point, and score the model based on how many pixels it classifies correctly.
2. Use an object-based approach, where we could an object as detected only if it is sufficiently covered by the prediction.  Predictions that did not find objects are false positives.

The first approach is really turning the problem into a segmentation problem, which is a different thing from detection.  That's not ideal for this case, because it makes large objects more important than small objects.  If we're detecting aircraft, we want to weigh large aircraft like airliners equally with small aircraft like fighter jets.  So segmentation won't do.

The second approach makes more sense for us, but there are a few details to work out.  First, how do we decide whether or not an object is detected.

* How accurate does the detection box have to be before we call an object detected?
* Do we care if the boxes are oversized?
* Can one box detect more than one object?
* How should we count false positives?

#### Hitting the target
![alt text][offset_predictions]

Which of these images is a good detection?  We don't want to be too picky about the exact location of objects, since its not really important in most applications, but we have to draw the line somewhere.  A simple way to deal with this is to set a threshold on the proportion of the object area covered by the prediction.  We can set a minimum of 0.4 to make sure we got most of the object in the image.

#### Box Size
![alt text][sized_predictions]

What about boxes that contain an entire object, but are much larger than the actual object?  Is it even a valid detection if we have a ridiculously oversized box like the one we looked at before? We want the boxes to represent the size of the object, and we'd like to avoid a scenario where a model gets rewarded for using big, imprecise boxes to increase it's hit rate.  So let's introduce the **Jaccard Index**, also known as **intersection over union** (**IoU**).

Before computing the **Jaccard index**, we'll switch from using the precise object markings to using a box circumcscibing the object because this gives better results.  

![alt text][sized_predictions_jacc]  
From looking at these images, it looks like a **Jaccard index** over 0.45 is a good detection.  A number below that is a sign that the box is too big or too small.  We can use a Jaccard cutoff instead of a proportion cutoff.


#### Multiple Detections


![alt text][2pred2] &nbsp;&nbsp; <font size="+3"> <b> or </b> </font> &nbsp;&nbsp;
![alt text][2pred3] 


Should we let a single box detect multiple airplanes?  If you just want to find airplanes, you might not care whether a box touches one plane or more than one.  But if your goal is to count planes, then you really want to enforce strict correspondence between boxes and planes.  If you want to restrict boxes to planes, then you can enforce that rule in the error calculation. My error calculator supports both.

#### False Positives
![alt text][total_miss]
![alt text][double_down] 

The picture on the left is clearly a miss.  The box just missed the target.  But what about the picture on the right?  Should we count both of those as hits, or count just one?  Or should we discount the overlapping portion in some way?

## Conclusions

I made my own decisions on these questions, and wrote a calculator that scores models based on my criteria.  In some cases, I supported more than one option so we could tailor our error measurement to specific problems.



[graveyard_plain]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/graveyard_plain.png "A section of the airplane graveyard"
{: width="400px"}
[graveyard_marked]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/graveyard_marked.png "A section of the airplane graveyard, marked with polygons"
{: width="400px"}
[bad_prediction]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/bad_prediction.png "A bad prediction that catches all of the airplanes."
{: width="400px"}
[offset_predictions]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/offset_predictions_prop.png "Predictions at different offsets"
{: width="1000px"}
[sized_predictions]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/sized_predictions.png "Predictions at different sizes"
{: width="1000px"}
[sized_predictions_jacc]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/sized_predictions_jacc.png "Predictions at different sizes"
{: width="1000px"}


[2pred2]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/2pred2.png "Two targets, one match"
{: width="200px"}
[2pred3]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/2pred3.png "Two targets, two matches"
{: width="200px"}

[total_miss]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/total_miss.png "This prediction missed the target completely."
{: width="400px"}
[double_down]: {{ site.baseurl }}/assets/images/2017-04-11-Measuring_Performance/double_down.png "Here we have two predictions that hit the target."
{: width="400px"}