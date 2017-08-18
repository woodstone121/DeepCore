---
layout: post
title: Introducing the Metis/DigitalGlobe Data Challenge
author: Alan J. Schoen
author_title: Data Scientist
desc: DigitalGlobe is teaming up with Metis to create a geospatial data science challenge.
keywords: data science, geospatial, machine learning, coding bootcamps, education
published: True
---

Last year, I completed the [Metis Data Science Bootcamp](https://www.thisismetis.com/data-science-bootcamps) in New York. I finished hiking the entire Appalachian Trail at the end of 2015, then spent 6 grueling months looking for a job. Once I accumulated a few hundred rejection letters, and it seemed like I wasn't any closer to a job, I realized my skills were outdated and I had to do something about it. I looked into my options: coding bootcamps, graduate school, and online graduate programs. I decided a coding bootcamp was the fastest and most economical way to polish my skills.

I previously worked in data science and computational modeling, but I used Matlab as my main programming language. At Metis, I learned about Python tools for data science, and how to present my work well. About two weeks after graduating, I was fortunate to find my current job at DigitalGlobe as a Data Scientist. Now, about 6 months later, I'm working with Metis to introduce more students to geospatial data and satellite imagery.

Metis students finish the coding bootcamp by completing a 3-week capstone “passion project”, in which they present a data science-based solution or application to an issue of their choosing. The curriculum prepares students for the project by developing skills like data acquisition, data wrangling, statistics, and model-building. DigitalGlobe provides data and tools for Metis students who want to work with satellite images for their passion project.

# Project Description
This project encourages students to find a way to detect land types pixel-by-pixel. This solution will let us use machine learning for tasks like land-cover classification and land-use classification, which are two important ways we get valuable information from satellite images.

There are many public sources of land cover data, like [GlobCover](http://due.esrin.esa.int/page_globcover.php), [NLCD](https://pubs.usgs.gov/fs/2012/3020/fs2012-3020.pdf), and [others](https://landcover.usgs.gov/uslandcover.php).  These are large projects published by government agencies.  They are great resources, but lack the resolution needed for a lot of applications.


When people need more accurate data, they often have to produce it themselves.  For example, The Bill & Melinda Gates Foundation recently completed a census estimate in Nigeria using satellite imagery and machine learning ([described here](https://www.nature.com/news/satellite-images-reveal-gaps-in-global-population-data-1.21957)).  They sent out researchers to collect survey data, and then used that data to calibrate computer models to extend their estimates to more areas.  Our goal in this project is to use a similar methodology to train models to recognize objects and terrain that we are interested in.  Instead of collecting data door-to-door, we will try to use publicly-available data from the internet.


![Population Estimate from the Gates Foundation (source: Nature Magazine)]({{ site.baseurl }}/assets/images/Metis_Project/nature-population-map-11-may-17-online.jpg){: width="40%"}


In order to complete this project, we are using a machine learning modality called segmentation.  Segmentation is related to the more commonly known task of classification, but differs in important ways.  Most people are familiar with image classification as an application of deep learning, in which categories are assigned to pictures. For example, a classifier looks at images and tells you whether each image contains an object of interest, like an airplane. Segmentation goes a step further and indicates where the airplane is in the image.


![Image Segmentation]({{ site.baseurl }}/assets/images/Metis_Project/segmentation.png){: width="40%"}


Everyone knows about the ImageNet classification challenge, but there are also challenges for segmentation.  [PASCAL VOC](http://host.robots.ox.ac.uk/pascal/VOC/) is the most well-known, and the PASCAL VOC dataset is still the go-to for training segmentation networks. In recent years, ImageNet itself has included a segmentation challenge.  There are also a [variety](http://biomedicalimaging.org/2017/challenges/) of [segmentation challenges](http://www.isles-challenge.org/) in the [medical domain](http://brainiac2.mit.edu/isbi_challenge/).


The idea for the Metis/DigitalGlobe Data Challenge is to create detailed maps by using segmentation on satellite images.  If we use segmentation to find water, for example, we can turn satellite images into maps of water, which has all kinds of applications. For instance, if you applied a water segmentation algorithm to the same images of coastline over and over again, you could track sea level changes. Or you could find flooded areas after a natural disaster.  If you consider other terrain types, like forest, farmland, and urban areas, there are many possibilities.


This is somewhat similar to a couple of [Kaggle](https://www.kaggle.com/c/planet-understanding-the-amazon-from-space) [competitions](https://www.kaggle.com/c/dstl-satellite-imagery-feature-detection).  The difference is that our goal is to have the students work out the *entire* project pipeline from data acquisition to results. So we are not handing over a dataset with images and ground truth; we are handing over a set of tools to help students acquire their own ground truth.  Each student will choose their own project, and we are hoping that they will come up with lots of interesting ideas.


# Project Materials

The main way I am supporting this project is by providing tutorials on [GitHub](https://github.com/DigitalGlobe/poirot). I named the repository after a TV detective (perhaps the greatest TV detective) in keeping with the Metis convention for naming projects. For the uninitiated, Hercule Poirot is basically the [Belgian James Bond](https://www.google.com/search?q=Hercule+Poirot&tbm=isch).  He is based on a recurring character in Agatha Christie's detective novels.

My tutorials will show how to train a neural net to classify residential areas, forests, and water in Paris (yes, there are forests in Paris).


## 0. Geospatial tools in python
Working with geospatial data is a little different from working with other kinds of data. Spatial is special, as they say. You need a special set of tools to work with geospatial data in Python, like GDAL, rasterio and GeoPandas. These packages can be hard to set up and hard to use. I provided a couple of notebooks to help with the system setup. [Ubuntu](https://github.com/DigitalGlobe/ggd-poirot/blob/master/Setup%20(Ubuntu).ipynb) [Mac](https://github.com/DigitalGlobe/ggd-poirot/blob/master/Setup%20(Mac).ipynb)


## 1. SpaceNet Images
[Notebook #1](https://github.com/DigitalGlobe/ggd-poirot/blob/master/1.%20Get%20Started%20With%20SpaceNet%20Images.ipynb) in the poirot repository shows how to download and start working with the SpaceNet data.


In order to train models, we will need a collection of satellite images to use.  For that, we are using the SpaceNet dataset, a public dataset collaboratively published by CosmiQ Works, DigitalGlobe, and NVIDIA. This is a collection of high-resolution imagery from five cities: Rio de Janeiro, Paris, Vegas, Khartoum, and Shanghai. DigitalGlobe released the Rio de Janeiro data for research purposes, and subsequently used it to launch our first open data challenge on TopCoder in 2016. We expanded the dataset to four more cities for the [second challenge](https://crowdsourcing.topcoder.com/spacenet).  


Most of the imagery is 30 cm resolution from our WorldView-3 satellite, except the Rio de Janeiro images, which are 50 cm from WorldView-2. In satellite imagery, resolution has a slightly different meaning than in most images. The resolution refers to the size of each pixel. For example, in 30 cm imagery, each pixel is 30 cm x 30 cm square. There are, of course, some variations around that resolution depending on the [cartographic projection](https://xkcd.com/977/), but let's not complicate things too much.


The tutorial materials work with image chips that were cut out of larger images.  The chipping was done as part of the preprocessing that CosmiQ Works did to prepare the data for machine learning.  DigitalGlobe images start out as image strips, which are giant multiple-gigabyte images covering many square kilometers. I have tools to process images like that, but they can be a little tricky to work with, so I left them out of the tutorials.

Here's a summary of the SpaceNet chips across the 5 cities.

|City|# of images|Resolution|Area covered (km2)|
|:-|:-|:-|:-|
|Rio|29|50 cm|2907|
|Vegas|3851|30 cm|135|
|Paris|1148|30 cm|38|
|Shanghai|4582|30 cm|543|
|Khartoum|1012|30 cm|40|

The area covered in Rio de Janeiro (2,907 km2) is larger than the actual city (1,260 km2).  Also, the number of images for Rio de Janeiro is much lower than the other images because it has a different structure. It is a giant mosaic broken up into 29 pieces. The other cities have 200 m x 200 m (650 px x 650 px) tiles.


This is what the coverage in Paris looks like:


![SpaceNet coverage in Paris]({{ site.baseurl }}/assets/images/Metis_Project/paris_coverage.png){: width="90%"}


And here's an example of one of the tiles in Paris:


![Sample tile in Paris)]({{ site.baseurl }}/assets/images/Metis_Project/paris_tile.png){: width="40%"}


## 2. Ground truth
[Notebook #2](https://github.com/DigitalGlobe/ggd-poirot/blob/master/2.%20Download%20Ground-Truth%20Data%20from%20OSM.ipynb) in the poirot repository shows how to download and process OSM data.


We train models by showing them a series examples of images that we already marked up. We could mark those images by hand, but that would be a lot of work. It is much easier if we can find a source of data to use; we call this ground truth. We suggest students use OpenStreetMap to get ground truth data.  OpenStreetMap is a giant, open, editable map of the world and it includes data like streets, buildings, and natural features. At DigitalGlobe, we value OpenStreetMap because it is the most extensive public database of spatial data. The [OpenStreetMap Wiki](http://wiki.openstreetmap.org/wiki/Map_Features) shows you the sheer range of object categories that are in OpenStreetMap. Unfortunately, the data can be inaccurate and incomplete, especially for the rare categories that people do not use often, so you need to make sure the data you want is good enough before you use it.

You can get OSM data by downloading it from OSM (as shown in notebook #2), or you can get the same information packaged in files from [GEOFABRIK](http://www.geofabrik.de/).  In addition, AWS [recently announced](https://aws.amazon.com/about-aws/whats-new/2017/06/openstreetmap-public-data-set-now-available-on-aws/) that they will be serving OSM data as part of their public datasets program.

## 3. Image Masks
[Notebook #3](https://github.com/DigitalGlobe/ggd-poirot/blob/master/3.%20Make%20training%20masks%20from%20OSM%20data.ipynb) in the poirot repository shows how to turn OSM data into training masks.

There is one more thing we need to do to the images before we can train a neural net: they need to be converted into masks. This is where all the complicated reprojection stuff comes into play. Fortunately, the python tools for geospatial data make this pretty easy. Follow the notebook to understand how.

## 4. Training a neural net
[Notebook #4](https://github.com/DigitalGlobe/ggd-poirot/blob/master/4.%20Train%20a%20Model.ipynb) in the poirot repository shows how to train a U-Net segmentation neural network.


Finally, after you have followed the first three notebooks, you are ready to train a neural net. I used Keras and Tensorflow to train a U-Net network. You will need a powerful GPU to train neural networks, and even then it is slow going.

There are a lot of great segmentation networks out there, like [FCN](https://github.com/shelhamer/fcn.berkeleyvision.org), [Mask R-CNN](https://arxiv.org/abs/1703.06870), and [ResNet](https://github.com/DrSleep/tensorflow-deeplab-resnet), but I thought [U-Net](https://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/) would be best for this project. U-Net is known for performing well when trained from scratch on small datasets, and you can get decent performance without having to solve all the challenges of transfer learning. Sadly, even U-Net was not able to learn much from the Paris dataset alone. To get a good model, it will be necessary to get more data, augment the current data, or use transfer learning.

# Update 6/2/17
The first group of Metis students is just about to finish their projects with the Metis/DigitalGlobe challenge.  Three students in the New York Spring 2017 cohort took on the challenge.  I have seen some preliminary results, and they look great.  The students will give their final presentations on Thursday, June 20th.

