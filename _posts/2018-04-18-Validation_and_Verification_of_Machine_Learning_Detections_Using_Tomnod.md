---
layout: post
title: Validation and Verification of Machine Learning Detections using Tomnod
author: Kevin McGee, Joe White
author_title: Production Lead, Software Development Manager
---

In our blog post entitled [Discovering Pattern of Life Activity using Machine Learning](http://radiantsolutions.com/blog/discovering-pattern-of-life-activity-using-machine-learning), we described how the output from a machine learning algorithm can aid in characterizing human activity over time. We did this by counting all the objects from certain categories like planes, trains and automobiles and then view the results in aggregate. This technique gives a solid activity baseline and can be used with other data sources like economic, demographic or market information.

In order to do all of this, we need machine learning models that perform well in all geographic areas.  Additionally,  we need a generous amount of training data to cover all the challenges we encounter when performing machine learning on satellite imagery. One option to accomplish this would be to have an army of imagery analysts marking our imagery all the time for our areas and objects of interests. Obviously this does not scale well in terms of cost or time. At Radiant Solutions we asked the question, “what if we could get our machine learning model to train itself?” So that’s exactly what we did. With our crowdsourcing platform [Tomnod](https://www.tomnod.com), we’ve developed an end-to-end pipeline to produce highly accurate models, as efficiently as possible.

![Workflow]( {{ site.baseurl }}/assets/images/Validation_and_Verification/workflow.png){: width="100%"}

Let’s start from the beginning. Let’s say we want to detect cars in satellite imagery. First we need to generate training data for cars. We can easily run a Tomnod “Discovery” campaign to create our initial training data corpus.  Discovery campaigns harness the crowd to find objects in imagery.  For example, a discovery campaign would allow users of Tomnod to locate cars in satellite images, which would rapidly provide many locations for training chips that could then be used to train a model.

![Tomnod Discovery]( {{ site.baseurl }}/assets/images/Validation_and_Verification/tomnod.png ){: width="100%"}

A general rule of thumb is that the more training data you have, the more accurate your model will be. We’re talking about hundreds of thousands of training samples to ensure your model is robust and accurate. With that said, if you don’t have the resources to produce that much training data, we’ve found that having at least 1,000 samples of an object is the point where most models start to learn. Then we can use our end-to-end pipeline to take care of the rest.

Next up, you’ll want to train your model. We’ll need to split off some of our training data into validation data so we can gauge how our model is improving over time. An 80/20 split (80% training data, 20% validation data) is a good start. We tend to ensemble our network architectures so it’s important to keep up and try different approaches to improve model performance.

Once we have our model, it’s time to start detecting objects. At Radiant Solutions we use [OpenSpaceNet](https://github.com/DigitalGlobe/GGD-OpenSpaceNet), which is powered by our machine learning abstraction framework [DeepCore](http://deepcore.io). OpenSpaceNet removes a lot of the complexities of running machine learning at scale, while still leaving the user the ability to tweak detection parameters as needed.

Here’s where the magic of a “model that trains itself” comes in. We use the output detections of the model as an input that goes back into the retraining of the model. This process has huge advantages to getting a highly accurate model, very quickly. First, this will reduce the amount of time needed to generate training data. If we have a model that has 50% accuracy, this essentially will reduce the amount of time needed to mark training data by 50%. This is because all we’re doing is validation and verification of the results from the model. We use our Tomnod API to automatically create a campaign when model results are ready which then asks a simple “yes” or “no” question about the detection.

![Tomnod V&V]( {{ site.baseurl}}/assets/images/Validation_and_Verification/tomnod2.png ){: width="100%"}

Another huge advantage is that the model can learn from its mistakes. Not only are we feeding valid detects back in for training, we are also feeding in invalid detections as negative training data. So if our model is picking up a lot of building corners, or trucks and labeling them as cars, we can quickly correct the model to prevent these mistakes.

One final piece to the puzzle is building a “generalizable” model. Machine learning on satellite imagery is hard because the earth is hugely diverse. On images taken on different days, a car in the same spot can look very different depending on things like the angle of the sun, or the resolution of the image. Also, cars in Europe are almost guaranteed to look different than cars in the U.S. just based off the differences in infrastructure in the world. So when we’re selecting new AOIs to run our models on we need to take this diversity into account. Essentially it boils down to three sets of diversities: geographic, seasonal, and imagery. For geographic diversity we want to ensure that our training data is evenly spread throughout the entire globe. For seasonal, we want to make sure that we have training data that accounts for all seasons, for instance snow covered cars as well as cars in the spring. Finally we want to make sure our training data accounts for all of the complexities of satellite imagery like resolution, sun and nadir angles, cloud coverage, etc.

All of these considerations come together in our end-to-end pipeline, where we start with only the task of “find cars in satellite imagery” to a highly accurate, generalizable machine learning model that can produce results like this:

![Detections]( {{ site.baseurl }}/assets/images/Validation_and_Verification/detects.png){: width="100%"}

Thanks for reading! If you’d like more information specifically about how we use Amazon’s Mechanical Turk with Tomnod to make this solution highly scalable please check out [this](https://youtu.be/prpDfnguAY8) presentation given at AWS re:Invent.