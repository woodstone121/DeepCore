---
layout: post
title: Introducing the DigitalGlobe/Metis Data Challenge
author: Alan J. Schoen
desc: DigitalGlobe is teaming up with Metis to create a geospatial data science challenge.
keywords: data science, geospatial, machine learning, coding bootcamps, education
published: True
---

Last year, I completed the Metis Data Science Bootcamp.  I had finished hiking the entire Appalachian Trail at the end of 2015, and then spent 6 grueling months looking for a job.  Once I had accumulated a few hundred rejection letters, and it seemed like I wasn't any closer to a job, I realized that my skills were outdated and I had to do something about it.  I looked into my options: coding bootcamps, graduate school, online graduate programs, and decided that a coding bootcamp was the fastest and most economical way to polish my skills.

I had already worked in data science and computational modeling, but I had used Matlab as my main programming language for my entire professional career. At Metis, I learned about Python tools for data science, and I learned about how to present my work better.  About two weeks after graduating, I was fortunate to find my current job at DigitalGlobe.  Now, about 6 months later, I'm working with Metis to introduce more students to geospatial data and satellite imagery.

Metis students finish the program by completing a 3-week passion project.  The entire curriculum up until that point prepares students to take it on by developing skills like data acquisition, data wrangling, statistics, and model-building.  We partnered with Metis to provide data and tools for students who want to work with satellite images for their passion project.

# Project Description
We're exploring how to segment land types pixel-by-pixel in this project.  This will let us use machine learning for tasks like land-cover classification and land-use classification.  When we're getting value out of satellite images, this is one of the main ways to do it.  
There are many public sources of land cover data, like [GlobCover](http://due.esrin.esa.int/page_globcover.php), [NLCD](https://pubs.usgs.gov/fs/2012/3020/fs2012-3020.pdf), and [others](https://landcover.usgs.gov/uslandcover.php).  These are large projects published by government agencies.  They are great resources, but they lack the resolution we need for a lot of applications.

When people need more accurate data, they often have to produce it themselves.  The Bill & Melinda Gates Foundation recently completed a census estimate in Nigeria using satellite imagery and machine learning ([described here](https://www.nature.com/news/satellite-images-reveal-gaps-in-global-population-data-1.21957)).  They sent out researchers to collect survey data, then they used that data to calibrate computer models to extend their estimates to more areas.  Our goal in this project is to use a similar methodology to train models to recognize objects and terrain that we're interested in.  Instead of collecting data door to door, we'll try to use publicly-available data from the internet.

![Population Estimate from the Gates Foundation (source: Nature Magazine)]({{ site.baseurl }}/assets/images/Metis_Project/nature-population-map-11-may-17-online.jpg){: width="40%"}


In order to complete this project, we're using a machine learning modality called segmentation, which is related to the more commonly known task of classification, but differs in important ways.  Most people are familiar with image classification as an application of deep learning.  Classification is the challenge of assigning categories to pictures.  For example, a classifier could look at 1,000 pictures and tell you whether or not each one contains an airplane.  Segmentation goes a step further and outlines the plane in the image.

![Population Estimate from the Gates Foundation (source: Nature Magazine)]({{ site.baseurl }}/assets/images/Metis_Project/segmentation.png){: width="40%"}

Everyone knows about the ImageNet classification challenge, but there are also challenges for segmentation.  [PASCAL VOC](http://host.robots.ox.ac.uk/pascal/VOC/) is the most well known, and the PASCAL VOC dataset is still the go-to for training segmentation networks.  In recent years, ImageNet itself has included a segmentation challenge.  There are also a [variety](http://biomedicalimaging.org/2017/challenges/) of [segmentation challenges](http://www.isles-challenge.org/) in the [medical domain](http://brainiac2.mit.edu/isbi_challenge/).

The idea for this project is to create detailed maps by using segmentation on satellite images.  If we use segmentation to find water, for example, we can turn satellite images into maps of water.  This has all kinds of applications.  For instance, if you applied a water segmentation algorithm to the same images of coastline over and over again, you could track sea level changes.  Or you could find flooded areas after a natural disaster.  If you consider other terrain types, like forest, farmland, and urban areas, there are a lot of possibilities.

This is somewhat similar to a couple of [Kaggle](https://www.kaggle.com/c/planet-understanding-the-amazon-from-space) [competitions](https://www.kaggle.com/c/dstl-satellite-imagery-feature-detection).  The difference is that our goal is to have the student work out the *entire* project pipeline from data acquisition to results.  So we aren't just handing over a dataset with images and ground-truth; we're handing over a set of tools to help students acquire their own ground-truth.  Each student will choose their own project, and we're hoping that they will come up with lots of interesting ideas.

# Project Materials

The main way I'm supporting this project is providing tutorials on [GitHub](https://github.com/DigitalGlobe/poirot).  I named the repository after a TV detective (perhaps the greatest TV detective) in keeping with the Metis convention for naming projects.  For the uninitiated, Hercule Poirot is basically the [Belgian James Bond](https://www.google.com/search?q=Hercule+Poirot&tbm=isch).

My tutorials will show how to train a neural net to classify residential areas, forests, and water in Paris (yes, there are forests in Paris).

## 0. Geospatial tools in python
Working with geospatial data is a little different from working with other kinds of data.  Spatial is special, as they say.  So you need a special set of tools to work with geospatial data in Python, like GDAL, rasterio and GeoPandas.  These packages can be hard to set up and hard to use.  I provided a [couple](https://github.com/DigitalGlobe/poirot/blob/master/Setup%20(Mac).ipynb) of [notebooks](https://github.com/DigitalGlobe/poirot/blob/master/Setup%20(Ubuntu).ipynb) to help with the system setup.

## 1. SpaceNet Images
[Notebook #1](https://github.com/DigitalGlobe/poirot/blob/master/1.%20Get%20Started%20With%20SpaceNet%20Images.ipynb) in the poirot repository shows how to download and start working with the SpaceNet data.

In order to train models, we'll need a collection of satellite images to use.  For that, we're turning to the SpaceNet dataset, a public dataset published by DigitalGlobe.  This is a collection of high-resolution imagery from 5 cities: Rio, Paris, Vegas, Khartoum, and Shanghai.  We originally published it as a resource for people coordinating the Rio Olympics.  We also used it to launch our first open data challenge on TopCoder in 2016.  Then we expanded the dataset to four more cities for the [second challenge](https://crowdsourcing.topcoder.com/spacenet) on TopCoder.  

Most of the imagery is at 30cm resolution from our WorldView 3 satellite, except the Rio images, which are 50cm from WorldView 2.  In satellite imagery, resolution has a slightly different meaning than in most images.  The resolution refers to the size of each pixel, so in 30cm imagery each pixel is a 30cm X 30cm square.  There are, of course, some variations around that resolution depending on the [cartographic projection](https://xkcd.com/977/), but let's not complicate things too much.

Here's a summary of the SpaceNet dataset across the 5 cities.

|City|# of images|Resolution|Area covered (km2)|
|:-|:-|:-|:-|
|Rio|29|50 cm|2907|
|Vegas|3851|30 cm|135|
|Paris|1148|30 cm|38|
|Shanghai|4582|30 cm|543|
|Khartoum|1012|30 cm|40|

The area covered in Rio (2,907 km2) is larger than the actual city of Rio (1,260 km2).  Also, the number of images in Rio is much lower than the other images.  This is because Rio has a different structure.  Rio is a giant mosaic that was broken up into 29 pieces.  The other cities have 400m x 400m (650px x 650px) tiles.

This is what the coverage in Paris looks like:

![SpaceNet coverage in Paris]({{ site.baseurl }}/assets/images/Metis_Project/paris_coverage.png){: width="90%"}

And here's an example of one of the tiles in Paris:

![Sample tile in Paris)]({{ site.baseurl }}/assets/images/Metis_Project/paris_tile.png){: width="40%"}


## 2. Ground-truth
[Notebook #2](https://github.com/DigitalGlobe/poirot/blob/master/2.%20Download%20Ground-Truth%20Data%20from%20OSM.ipynb) in the poirot repository shows how to download and process OSM data.

We train models by showing them a series examples of images that we already marked up.  We could mark those images by hand, but that would be a lot of work.  It's much easier if we can find a source of data to use.  We call this ground truth.  We suggest that students use OpenStreetMap to get ground truth data.  OpenStreetMaps is a giant open database of map data including streets, buildings, and natural features.  At DigitalGlobe, we value OpenStreetMaps a lot, because its the most extensive public database of spatial data.  The [OpenStreetMaps Wiki](http://wiki.openstreetmap.org/wiki/Map_Features) shows you the sheer range of object categories that are in OpenStreetMaps.  Unfortunately, the data can be inaccurate and incomplete, especially for the rare categories that people don't use often, so you do need to make sure the data you want is good enough before you use it.

You can get OSM data by downloading it from OSM, or you can get the same information packaged in files from [GEOFABRIK](http://www.geofabrik.de/).

## 3. Image Masks
[Notebook #3](https://github.com/DigitalGlobe/poirot/blob/master/3.%20Make%20training%20masks%20from%20OSM%20data.ipynb) in the poirot repository shows how to turn OSM data into training masks.

There's one more thing we need to do to the images before we can train a Neural Net, they need to be converted into masks.  This is where all the complicated reprojection stuff comes into play.  Fortunately, the python tools for geospatial data make this pretty easy.  Follow the notebook to see how.

## 4. Training a neural net
[Notebook #4](https://github.com/DigitalGlobe/poirot/blob/master/4.%20Train%20a%20Model.ipynb) in the poirot repository shows how to train a U-Net segmentation neural network.

Finally, after you have followed the first three notebooks, you're ready to train a neural net.  I used Keras and Tensorflow to train a U-Net network.  There are a lot of great segmentation networks out there, like [FCN](https://github.com/shelhamer/fcn.berkeleyvision.org), [Mask R-CNN](https://arxiv.org/abs/1703.06870), and [ResNet](https://github.com/DrSleep/tensorflow-deeplab-resnet), but I thought [U-Net](https://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/) would be the best starting point for this project.  That's because U-Net is known for performing well when trained from scratch on small datasets.  That makes it a good starting point, because you can get decent performance without having to solve all the challenges of transfer learning

You will need a powerful GPU to train neural networks, and even then it's slow going.