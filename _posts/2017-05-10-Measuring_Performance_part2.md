---
layout: post
title: Measuring Performance for Object Detectors - Part 2
author: Alan J. Schoen
desc: This post is about measuring accuracy of object detection neural network models on geospatial imagery.
keywords: model accuracy, model validation, machine learning metrics
published: True
---

This post is the second and final installment in my series about measuring the accuracy of models.  In my last post, I talked about how to choose a method to score models.  Now it's time to use that method to score models against each other.  Let's start with a brief summary of the goal.

We have a ground truth image, in which each airplane has been painstakingly marked by an intern in the DigitalGlobe offices:

![Airplane Graveyard]({{ site.baseurl }}/assets/images/Measuring_Performance_2/graveyard_marked.png){: width="70%"}

A detection model's goal is to find all of the airplanes in an image by drawing a box around each instance.  A perfect prediction would look like this:

![A perfect prediction]({{ site.baseurl }}/assets/images/Measuring_Performance_2/perfect_prediction.png){: width="65%"}

Any red stripy areas left over would be bad, because they mean the model missed something.  This prediction is perfect, so it didn't miss anything.  No real model is going to perform that well, but the DetectNet model can at least come close.  I ran the DetectNet model on this image, and now I'll go over the results.  You may recall that I bragged about this model in a [previous post](https://digitalglobe.github.io/DeepCore/2017/04/26/Creating_Synthetic_Clouds.html).

### Using all of the model predictions: threshold 0.0

![Airplane Graveyard]({{ site.baseurl }}/assets/images/Measuring_Performance_2/prediction9.png){: width="65%"}
&nbsp;&nbsp;&nbsp;
![Airplane Graveyard]({{ site.baseurl }}/assets/images/Measuring_Performance_2/hist9.png){: width="22%"}

On the left, you'll see an image with all of the detection boxes superimposed, and on the right you can see a histogram of the model scores, with the threshold (0) marked with an orange line.  That's an awful lot of detection boxes.  The prediction found all 33 airplanes in the image (true positives), but there are also 4339 unnecessary detection boxes (false positives).  To understand the model performance better, it's useful to introduce two concepts: [precision and recall](https://en.wikipedia.org/wiki/Precision_and_recall).  Recall is the percentage of targets found by the model, it is computed by dividing the number of true positive by the number of targets.  Precision is the percentage of prediction boxes that actually found an object, and it is computed by dividing the number of true positives by the total number of detection boxes that the model placed.  A common way to score models is with the F1 score, which combines precision and recall by taking the harmonic mean.

![F1 Formula]({{ site.baseurl }}/assets/images/Measuring_Performance_2/f1.png){: width="22%"}

The harmonic mean is useful here because the `F1` score can only be large if both the `precision` and `recall` are large.  There is also a variation on this score that can increase the importance of recall (see the general formula for positive beta [here](https://en.wikipedia.org/wiki/F1_score)).  At this threshold, the model has an `F1` score of 0.015, which is not very good.

We can try to fix this by setting a higher threshold.

### Filter out low-confidence predictions: threshold 0.007
![Airplane Graveyard]({{ site.baseurl }}/assets/images/Measuring_Performance_2/prediction4.png){: width="65%"}
&nbsp;&nbsp;&nbsp;
![Airplane Graveyard]({{ site.baseurl }}/assets/images/Measuring_Performance_2/hist4.png){: width="22%"}

Now we have eliminated the very low-confidence predictions.  I also removed some of the low-confidence predictions from the histogram so that you could see the other scores better.  This prediction found 31 planes, but unfortunately it missed 2.  The model is not used to dealing with airplanes packed this close together.  It is quite rare, so I can't blame the model for struggling.  There are 10 false positives, most of which are duplicate detections.  There are also a few bonafide false positives in the upper left part of the picture.  At this threshold level, the model has an `F1` score of 0.84, which is pretty good.

### Narrowing it down more: threshold 0.593
I'm going to show one more variation that shows what happens when the threshold is too high.

![Airplane Graveyard]({{ site.baseurl }}/assets/images/Measuring_Performance_2/prediction3.png){: width="65%"}
&nbsp;&nbsp;&nbsp;
![Airplane Graveyard]({{ site.baseurl }}/assets/images/Measuring_Performance_2/hist3.png){: width="22%"}

At this threshold level, there are no false positives, but the model missed half of the planes.  We have only 16 true positives, out of a maximum of 33.  As a result, the model has an `F1` score of 0.65, which is lower than the last threshold.

I repeated the precision/recall/F1 analysis at 10 different threshold levels, and you can see the results in the table below:

|   Threshold |   True Positives |   False Positives |   False Negatives |   Precision |    Recall |     F1 |
|------------:|-----------------:|------------------:|------------------:|------------:|----------:|-------:|
| 0.908       |                1 |                 0 |                32 |  1          | 0.0303    | 0.0588 |
| 0.901       |                2 |                 0 |                31 |  1          | 0.0606    | 0.114  |
| 0.873       |                6 |                 0 |                27 |  1          | 0.182     | 0.308  |
| 0.593       |               16 |                 0 |                17 |  1          | 0.485     | 0.653  |
| 7.19e-03    |               31 |                10 |                 2 |  0.756      | 0.939     | 0.838  |
| 2.53e-04    |               33 |                72 |                 0 |  0.314      | 1         | 0.478  |
| 1.59e-05    |               33 |               234 |                 0 |  0.124      | 1         | 0.22   |
| 7.23e-07    |               33 |               645 |                 0 |  0.0487     | 1         | 0.0928 |
| 1.77e-08    |               33 |              1689 |                 0 |  0.0192     | 1         | 0.0376 |
| 1.89e-16    |               33 |              4337 |                 0 |  0.00755    | 1         | 0.0150 |
| 0           |               33 |              4339 |                 0 |  0.00755    | 1         | 0.0150 |

As we decrease the threshold, the number of true positives increases, but so does the number of false positives.  Eventually, the number of false positives gets out of control and rises into the thousands.  Looking in the `F1` column, you can see that the threshold of 7.19e-03 is indeed the best threshold we tested.  The standard way to look at these results is in a ROC curve.  Unfortunately, this problem doesn't fit well with that analysis because it requires converting the true positive and false positive measurements into rates.  With the true positives this is easy enough: divide the true positive count by the total number of targets.  But with false positives, it's not clear what we should use as the denominator.  So instead, we'll look at a precision recall-curve, since there is no obstacle to computing precision or recall.

![Precision/Recall Curve]({{ site.baseurl }}/assets/images/Measuring_Performance_2/precision-recall.png){: width="35%"}

Now you can see the trade-off that occurs at different thresholds.  As we lower the threshold the recall rate increases, but the precision also decreases.  Still, you can see parts of the curve where both precision and recall are high.  These are the areas where the `F1` score will also be high.

## Comparing to airplanes.gbdxm
For comparison, let's test out airplanes.gbdxm, which is available on the [DeepCore website](https://digitalglobe.github.io/DeepCore/index.html#five).  This is an AlexNet model that was trained on painstakingly hand-collected data.  As with the DetectNet model, I tested it at 10 different thresholds and picked the one with the highest `F1` score.

![Airplane Graveyard]({{ site.baseurl }}/assets/images/Measuring_Performance_2/prediction_test.png){: width="65%"}


This model had trouble detecting some planes, especially the small ones.  This is partly related to the window size.  This model can only make predictions at one size, so it's not capable of hitting the Jaccard cutoff (explained in my [previous post](https://digitalglobe.github.io/DeepCore/2017/04/26/Creating_Synthetic_Clouds.html)) for the really small aircraft.  Interestingly, all of the false positives in this example actually hit airplanes, but they were just too large to count as true positives.  This was also the case with the DetectNet model, but it was harder to see because the predictions were stacked on top of each other.  We could improve the performance of this model by running it a second time with a smaller window size, but that would make the model take even longer to compute and it's already a lot slower than the DetectNet model.

Now let's compare the two models in a table

|Model|True Positives|False Positives|False Negatives|Precision|Recall|F1|
|DetectNet|31|10|2|0.756|0.939|0.838|
|airplanes.gbdxm|23|4|10|0.852|0.697|0.767|

DetectNet performed better on recall and F1, but the airplanes model outperformed DetectNet on one measure: precision.  This is because airplanes.gbdxm has better non-maximum suppression.  We are working on improving our non-maximum suppression strategy for DetectNet models, and I expect it will outperform airplanes.gbdxm on precision too once that is finished.

