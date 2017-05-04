---
layout: post
title: Frontness&#58; Applications of Geospatial Object Detection
author: Ryan Desmond
tags: [frontness, applications, counting]
---

In my work on DeepCore, I have pleasure of being exposed to many of the applications that benefit from automated geospatial object detection. Machine learning is the key to unlocking data's potential, and some of the applications are completely intractable without automated techniques. The DeepCore team has worked hard to develop software for these problems. 

When Alan blogged about [measuring performance]({{ site.baseurl }}{% post_url 2017-04-11-Measuring_Performance_part1 %}) earlier in the month, he spoke about what makes a good model. He spoke at length about metrics and calculating them and about how the target application affects the design of the model and the metrics. Today, let's talk about a few simple applications and how deep learning impacts each one.


## Counting and Aggregation

One fundamental problem that can be solved in an automated way with machine learning is **object counting**. While many people have been approaching this problem in small scale, using the DeepCore framework in combination with [GBDX](http://platform.digitalglobe.com/gbdx/), allows it to be approached at global scale. This type of problem is to answer questions about *"how many"* 


In it's simplest form, the detector emits the center of objects. This is useful for derived intelligence such as a heatmap of activity (e.g. [*where are most the swimming pools?*](https://platform.digitalglobe.com/gbdx-poolnet-identifying-pools-satellite-imagery/)) or a graph showing change over time (e.g. [*where are people driving cars?*](https://medium.com/the-downlinq/car-localization-and-counting-with-overhead-imagery-an-interactive-exploration-9d5a029a596b)). With an automated system, it's easy to expand those queries over space and time (e.g. *how is the popularity of each airport in the United States changing?*).

![Car counting][car_counting]

That's a lot of utility from a scattering of points!  


### Improved Location Description

Expanding beyond simple points, a detector can output other characteristics of the object. From a remote sensing object detection perspective the next level of detection specificity improves on the description of the object's spatial location.
 
![Object Location Results][object_location_results]

While the current release of DeepCore only contains support for a center point or a geographic [bounding box](http://wiki.openstreetmap.org/wiki/Bounding_Box), research is ongoing to build models and add support for [other output paradigms](https://crowdsourcing.topcoder.com/spacenet). In a geographical application that involves relocatable objects, however, orientation can have more meaning than just finding a tighter fit.
 

### #Frontness

At this point, I've been resisting using a hashtag, I really have. But after a long discussion on important characteristics of the object to resolve, **#frontness** was born. While the only reference to "frontness" I can find online is as [jargon in phonetics](https://www.uni-due.de/ELE/Phonetics.htm), I take to it here to refer to the state or quality of an object's front. While "front" is an intuitive concept, I'll further elaborate on that as the "natural heading of an object".

A natural heading, however, isn't directly measurable. Ideally, a detection model would have an attribute that expresses orientation along with position. While a deep learning abstracts the details of obtaining this orientation, we have put some thought into other metrics that could be used to obtain it (perhaps as a metric to accompany the deep learning framework or in the development of a training set). Some of these metrics are:

![Frontness Surrogate Attributes][frontness_surrogate_attributes]

Unfortunately, these methods neither differentiate between front and back nor are they universally applicable. As modeling advances, a direct output would be the frontness. In the counting problem, models with #frontness can be used to infer the likely heading of each object. Why might this be useful?  An example could be measuring traffic. From overhead you can determine the density of the automobiles, but the measurement would have a great deal more value if we were to determine the direction of the traffic. Is it a beach day?  Are people evacuating?


## Search and Discovery

One of the hard problems posed to users of geospatial intelligence is the **needle in the haystack** problem. In this problem, you have to search for an object over a vast area. Let's work through an example: a plane has to make an emergency landing over the Australian outback. In such a remote area, it wasn't on anyone's radar when it landed, so it is up to the search and rescue team to find them. How can DeepCore help?  Once imagery is available over the region in which that the plane was lost, a detector is run over the area to quickly determine locations to search.

![Search and Discovery][needle_in_a_haystack]

A big challenge in this problem is developing a suitable model to perform this task. Since you are looking for something that is hard to find, it's necessarily hard to develop the breadth of comparable examples with which to train a model. With deep learning and convolutional neural networks, this problem is particularly acute. Some approaches that are useful here are:

 - Data Augmentation
 - [Transfer Learning](http://cs231n.github.io/transfer-learning/)
 - Synthesized Training Data
 - Training Methods (e.g. [dropout](http://jmlr.org/papers/v15/srivastava14a.html))

Those ideas are not unique to the search and discovery problem. For example, the well known network *AlexNet* has many millions of parameters that need to be learned and avoiding overfitting is always a challenge. As such, the discussion of building a model in the absence of sufficient data (which in some sense, you always are) will be featured in future posts.



[object_location_results]: {{ site.baseurl }}/assets/images/2017-05-02-Frontness/attributes.svg "Predictions at different sizes"
{: width="100%"}

[car_counting]: {{ site.baseurl }}/assets/images/2017-05-02-Frontness/points.svg "Car Counting"
{: width="100%"}

[frontness_surrogate_attributes]: {{ site.baseurl }}/assets/images/2017-05-02-Frontness/frontness.svg "Frontness Surrogate Attributes"
{: width="100%"}

[needle_in_a_haystack]: {{ site.baseurl }}/assets/images/2017-05-02-Frontness/australia.svg "Search and Discovery"
{: width="100%"}