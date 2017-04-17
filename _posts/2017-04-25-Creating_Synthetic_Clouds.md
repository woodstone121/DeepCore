---
layout: post
title: Creating Synthetic Clouds in Python
author: Alan J. Schoen
published: False
---



When we're training models to work on satellite imagery, it's important to build models that are resilient.  That means making models that can deal with the same variations we see in real sattelite imagery, like clouds, off-nadir angle, time of day, and atmospheric conditions.  I'm going to focus on clouds for this post.

I did a little research before righting this post.  You can find examples online where people [use graphics software](https://docs.gimp.org/en/python-fu-foggify.html) to [create clouds](http://smallbusiness.chron.com/create-perfect-clouds-gimp-37351.html).  However, most people are used to looking at clouds from ground level on planet earth, not from satellites in space.  So we should have a look at some clouds from sattelite images and then decide how to proceed

SHOW EXAMPLES

As you can see, clouds actually look pretty much the same from space.  File that under "Today I Learned".

I found a great [example](http://lodev.org/cgtutor/randomnoise.html) creating clouds in *gasp* C.  From looking at the source code, I can see that the author created some white noise, and then progressivly upsampled smaller and smaller parts of the image and then stacked the results on top of each other.

In psuedocode
1. generate and NxN white noise image and store it
2. cut out the upper-left quarter of the image
3. upsample the upper-left corner to the original image size, and store the result
4. repeat steps 2 and 3 until the corner image is 1x1
5. stack the original image, and all the stored upsampled images on top of each by adding the pixel values

I changed the order of the algorithm a little bit, but the result should be the same.  By varying the original image size, you can change the general appearance of the clouds.  Starting with a 64x64 image, the result looks like a coudy day.  Starting with 256x256, it looks like heavy rain.

We can reproduce this in Python.  I'm going to upgrade to cubic interpolation

```
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import scipy
import scipy.ndimage
% matplotlib inline

# Initialize
im_size = 768
base_pattern = np.random.uniform(0,255, (im_size, im_size))
turbulence_pattern = np.zeros((im_size, im_size))

# Create cloud pattern
power_range = range(2, int(np.log2(im_size)))
for i in power_range:
    subimg_size = 2**i
    quadrant = base_pattern[:subimg_size, :subimg_size]
    upsampled_pattern = scipy.misc.imresize(quadrant, (im_size, im_size), interp='bilinear')
    turbulence_pattern += upsampled_pattern / subimg_size

# Normalize values
turbulence_pattern /= sum([1 / 2**i for i in power_range])

# Plot the results
norm = matplotlib.colors.Normalize(vmin=0, vmax=255, clip=True)
plt.imshow(turbulence_pattern, cmap=plt.cm.gray, norm=norm)
```