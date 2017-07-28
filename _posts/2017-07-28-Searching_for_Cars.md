---
layout: post
title: Searching for Cars
author: Spencer Charney
---

I have spent the summer of 2017 as an intern, originally with the title of a UI/UX Developer, but my position soon changed. After a month of working on the project I was assigned to upon arrival, the team had completed its neccessary tasks. This allowed for the data science team to pick me up from my free agency to gain experience with machine learning.

# Newbie

My first encounter with machine learning began with training an image classification model in [DIGITS](https://developer.nvidia.com/digits), NVIDIA's GPU training system, to recognize hand-written digits. The training data used for the model was the [MNIST handwritten digit database](http://yann.lecun.com/exdb/mnist/), and the network was [LeNet-5](http://yann.lecun.com/exdb/lenet/). With just a few clicks, DIGITS allowed for me, who had no prior experience with machine learning, to create a dataset and begin training a model within minutes. The simplification of training models, without having to run scripts or dig through layers of a network, enabled me to spend my time learning about model training instead of worrying about pesky bugs or mistakes I would make when dealing with unfamiliar code.

![DIGITS Photo]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/digits.png){: width="100%"}

Following along the tailored tutorial for beginners to create the handwriting recognition model gave me a smooth introduction to the world of data science, and a tase of the capabilities of machine learning.


# Count the cars!

With the end goal of being able to successfuly count the amount of cars in a given image, I was given two different approaches to compare. One model would be created using object detection, and the other, semantic segmentation.

The output images of the models are very different, with the object detection network placing bounding boxes around the objects, and the segmentation model proudcing an image with white blobs. This can be problematic when trying to compare the two. Normal methods like Intersection over Union (IoU) don't work the same way when used with detection and segmentation, so I had to come up with different ways to measure the model accuracy so we could compare different model types. The object detection model will give an exact number of bounding boxes that it found in an image, but my big question is how can I accurately quanitify the results from a segmentation inference image? 


## DetectNet

In the first approach, I utilized an object detection network named [DetectNet](https://devblogs.nvidia.com/parallelforall/detectnet-deep-neural-network-object-detection-digits/). This network uses the [KITTI dataset format](http://www.cvlibs.net/datasets/kitti/raw_data.php), which has a specific file structure and labeling convention shown below. The first number on each line in the label indicates the class of the object marked, and in this instance 18 is a car.


![DetectNet file structure]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/detectnet_label.png){: width="70%"}


I had the opportunity to label my own dataset using a bounding box creator tool. It gave me perspective on how much time goes into creating a large enough dataset to train an accurate model. I marked close to 3,000 cars in 183 images and it took two full days to complete. That dataset was too small to train a successful model on, and I was given access to over 1600 images already labled by GIS technicians. 


The DetectNet network is not yet optimized to place bounding boxes around such small objects, such as the cars we are searching for. There is no clear answer I can give in my limited experience with the network and machine learning in general, but it is most likely due to the mixture of small training sets, tiny objects, and the confidence the model has in detecting said objects. Those with much more experience than I will be working in the near future to modify the network structure to see if DetectNet can be a viable option for car detection.

Once the network has been altered to be more effective at detecting the small objects we are looking for, counting the cars will be as simple as looking how many bounding boxes were placed on the inference images. 


## Semantic Segmentation

In the second approach, I used a fully-convolutional neural network for [semantic segmentation](https://github.com/NVIDIA/DIGITS/tree/digits-5.0/examples/semantic-segmentation#loading-the-data-into-digits). The datasets used to train a segmentation model consist of the original images and a folder of labeled images that contain masks of the objects the model needs to learn. The models that were trained with the labels filled completely with white had difficulty differentiating between cars that were close together. The bane of segmentation models is the bleeding that occurs when the blobs are larger than the objects, as it can make seeing multiple objects difficult due to there being one large blob covering them all. To reduce this issue I applied a gradient to the mask images so the model could have an easier distinction between cars that were close togther. 

![Different masks]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/gradient.png){: width="100%"}

Image on the left is without any alterations, and the image on the right is after applying the signed distance function.

The output of the model trained with the new label images made minor improvements upon it predecessor, but any improvement can mean the difference of an accurate or inaccurate estimation of cars.


![Original image]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/f712db010809.png){: width="33%"}
![Old model]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/oldmodel.png){: width="33%"}
![New model]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/newmodel.png){: width="33%"}

The image in the middle is the unchanged label model, while the image on the right is the one trained with the altered labels. The differences are difficult to see without zooming in on the images, but the elimination of excess bleeding will be essential for counting. 


One of the methods to count the number of cars based off of an inference image from a segmentation model that I used is to take the number of white pixels in the image and divide it by the average pixel area that a car occupies. I wrote a quick Python script to read over 1600 labels from the training data and determined that the average area of a car is 181 pixels. Taking the inference images and applying a small thresholding script using [OpenCV](http://docs.opencv.org/trunk/index.html) to make sure that all white pixels that are counted are a part of a car.


{% highlight python %}
import cv2
from glob import glob

def label(file):
    count = 0
    file = open(file, 'r')
    lines = file.readlines()
    for line in lines:
        if line[:2] == '18':
            count += 1
    return count

for fn in glob('masks/*.png'):
    im = cv2.imread(fn,0)
    ret, thresh = cv2.threshold(im, 45, 255, cv2.THRESH_BINARY)
    print("Estimated car count: " + str(cv2.countNonZero(thresh)/175) + " - " + str(cv2.countNonZero(thresh)/185))
    print("There are - " + str(label('labels/'+fn[6:-3]+'txt')) + " - labeled cars in the image: " + fn + "\n")
{% endhighlight %}

This script will simply output a few lines of information per inference image. It then will output the actual number of cars that were labeled in the image to determine the accuracy of the model's estimation.


```
Estimated car count: 85.48571428571428 - 80.86486486486487
There are - 83 - labeled cars in the image: masks/ffc423d9d709.png
```


This is just one of the many methods that can be utilized to try to count the number of objects based on a segmentation inference image. It will never be exact because it is based on averages and estimations, but it has potential to be a reliable approach to count the cars.

