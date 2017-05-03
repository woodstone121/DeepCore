---
layout: post
title: Measuring Performance for Object Detectors - Part 2
author: Alan J. Schoen
published: True
---

This post is the second and final installment in my series about measuring the accuracy of models.

In my last post, I talked about how to choose a method to score models.  Now it's time to use that method to score models against each other.  Let's start with a brief summary of the goal.

We have a ground truth image, in which each airplane has been painstakingly marked by an intern in the DigitalGlobe offices.

SHOW PICTURE

A model's goal is to find all of the airplanes in this image.  Which would look like this:

SHOW PICTURE

Any red stripey areas left over are bad, beacuse they mean the model missed something.  Let's start with airplanes.gbdxm, the model we distrubute on the [DeepCore website](https://digitalglobe.github.io/DeepCore/index.html#five).  This is an AlexNet model that was trained on (again) pain-stakingly hand-collected screenshots.

SHOW PICTURE

Now let's score this prediction.  I wrote a simple script to count the number of true positives, false positives, and false negatives.  Geopandas makes this pretty easy.

```
import geopandas as gpd
...
```

We can see that the model caught XX of the planes, but it missed a few of the little ones.


Now, moving on to another AlexNet model.  This time, instead of gathering screenshots by hand, our interns marked the location of planes in satellite images and then we used an automated system to create training chips.  This is an improvement because it standardizes the process, and lets us get the most out of each marked image.  For reasons of personal vanity, I call this network AlNet.

SHOW IMAGE

REPORT ACCURACY

Finally, let's look at the DetectNet model.  I have bragged about this model [in the past](https://digitalglobe.github.io/DeepCore/2017/04/26/Creating_Synthetic_Clouds.html).

SHOW IMAGE

REPORT ACCURACY

This is a huge improvement.

TABLE COMPARING ALL 3 MODELS

EXTRA (NOT NECESSARY)
The DetectNet model and AlNet have an extra feature in addition to detection.  They're both pretty good at classifying aircraft too.  Let's do one additional test where the models dont just have to find airplanes, they have to tell us what type they are too.  In this training data, we divided airplanes into 4 classes:

1. Airliners
2. Fighters
3. Military Cargo (includes bombers)
4. Other

We can make a small modification to the scoring script so that it's class-specific

```
show code
```

Now let's score the models

