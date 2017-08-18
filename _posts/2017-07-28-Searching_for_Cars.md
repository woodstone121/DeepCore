---
layout: post
title: Searching for Cars
author: Spencer Charney
author_title: Data Scientist Intern

---

This summer, I started at DigitalGlobe as an intern focused on UI/UX Development. After a month, my team completed our assignment. So I joined the data science team to gain experience with machine learning.

# Newbie

My first encounter with machine learning began with training an image classification model to recognize hand-written digits in [DIGITS](https://developer.nvidia.com/digits), NVIDIA's GPU training system. I used the [MNIST handwritten digit database](http://yann.lecun.com/exdb/mnist/) as training data for the model. The network was [LeNet-5](http://yann.lecun.com/exdb/lenet/). With just a few clicks, DIGITS allowed me, someone with no prior experience using machine learning, to create a dataset and begin training a model. The training models were simplified – I didn’t have to run scripts or dig through layers of a network. This enabled me to spend my time learning about training models instead of worrying about pesky bugs or mistakes I would make when dealing with unfamiliar code.

![DIGITS Photo]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/digits.png){: width="100%"}

Following the tailored beginners tutorial to create a handwriting recognition model gave me a smooth introduction to the world of data science, and a taste of the capabilities of machine learning.

# Count the cars!

I was challenged to create a model that would successfuly count the amount of cars in a given image. I used two different approaches to compare the results: object detection, and semantic segmentation.

The output images of the models are very different, with the object detection network placing bounding boxes around the objects, and the segmentation model producing an image with white blobs. This can be problematic when trying to compare the two. Normal methods like Intersection over Union (IoU) don't work the same way when used with detection and segmentation, so I had to come up with different ways to measure the model accuracy for comparison. The object detection model gives an exact number of bounding boxes found in an image. My big question is how can I accurately quanitify the results from a segmentation inference image? 

## DetectNet

In my first approach for counting cars, I used an object detection network named [DetectNet](https://devblogs.nvidia.com/parallelforall/detectnet-deep-neural-network-object-detection-digits/). This network uses the [KITTI dataset format](http://www.cvlibs.net/datasets/kitti/raw_data.php), which has a specific file structure and labeling convention shown below. The first number on each line in the label indicates the class of the object marked. In this instance, 18 is a car.


![DetectNet file structure]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/detectnet_label.png){: width="70%"}


I labeled my own dataset using a bounding box creator tool. It gave me perspective on how much time goes into creating a large enough dataset to train an accurate model. I marked 183 images containing close to 3,000 cars while I waited for access to prelabeled data. My manually-created dataset was too small to train a successful model on, but thankfully, soon afterward, I was given access to over 1600 images already labled by GIS technicians. This let me create new datasets that were much larger to improve the model's learning. 

The DetectNet network is not yet optimized for placing bounding boxes around small objects, like cars. In my limited machine learning experience, I would guess this is due to small training sets, tiny objects, and the confidence the model has in detecting said objects. Those with much more experience than I will be working in the future to modify the network structure to see if DetectNet can be a viable option for car detection.

Once the network is altered to be more effective at detecting the small objects, the goal of counting cars will be as simple as looking at how many bounding boxes were placed on the images. 


## Semantic Segmentation

In the second approach, I used a fully-convolutional neural network for [semantic segmentation](https://github.com/NVIDIA/DIGITS/tree/digits-5.0/examples/semantic-segmentation#loading-the-data-into-digits). The datasets used to train a segmentation model consist of the original images and a folder of labeled images that contain masks of the objects the model needs to learn. The models that were trained with the labels filled completely with white had difficulty differentiating between cars that were close together. The issue with segmentation models is the bleeding that occurs when the blobs are larger than the objects. It can make seeing multiple objects difficult because the results look like one large blob covering all the cars. To reduce this issue I applied a gradient to the mask images so the model could have an easier distinction between cars that were close togther. 

![Different masks]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/gradient.png){: width="100%"}

The image on the left above is without alterations. The image on the right above is after applying the gradient to the mask images.

The output of the model trained with the new label images made minor improvements upon its predecessor. Any improvement can mean the difference between an accurate or inaccurate estimation of cars.


The image on the left is the original image fed into the model. The image in the middle is the output of the unchanged label model, while the image on the right is the output of the model trained with altered labels. The differences are difficult to see without zooming in on the images, but the elimination of excess bleeding will be essential for counting. 

![Original image]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/f712db010809.png){: width="33%"}
![Old model]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/oldmodel.png){: width="33%"}
![New model]({{ site.baseurl }}/assets/images/2017-07-28-Searching_for_Cars/newmodel.png){: width="33%"}




To count the number of cars based off an inference image from a segmentation model, I took the number of white pixels in the image and divided it by the average pixel area that a car occupies. I wrote a quick Python script to read over 1600 labels from the training data and determined that the average area of a car is 181 pixels. I took the inference images and applied a small thresholding script using [OpenCV](http://docs.opencv.org/trunk/index.html) to make sure that all white pixels counted are a part of a car.


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

This script will output a few lines of information per inference image, before giving the actual number of cars that were labeled in the image to determine the accuracy of the model's estimation.


```
Estimated car count: 85.48571428571428 - 80.86486486486487
There are - 83 - labeled cars in the image: masks/ffc423d9d709.png
```


This is just one of many methods that can be used to count the number of objects based on a segmentation inference image. It will not be exact because it is based on averages and estimations, but any improvement in accuracy will improve the model's results while counting cars.


Having the opportunity to be a part of this project has pushed the boundaries of my knowledge of both computer science and machine learning. I am interesting in gaining more experience in this area because it is the future of computing. I was fortunate to have my first project wrapped up quickly so I could have the chance to be apart of the data science team. 

