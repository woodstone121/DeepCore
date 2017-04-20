---
layout: post
title: Creating Synthetic Clouds in Python
author: Alan J. Schoen
published: True
---

## Tiny Changes Can Fool AI
There has been a lot of [talk](https://motherboard.vice.com/en_us/article/fooling-image-classification-networks-is-really-easy) [recently](http://www.bbc.com/future/story/20170410-how-to-fool-artificial-intelligence) (and some [not so recently](https://motherboard.vice.com/en_us/article/machine-vision-google-adversarial-images)) about how tiny changes to pictures can fool that smartest neural nets.  One [paper](https://www.cs.cmu.edu/~sbhagava/papers/face-rec-ccs16.pdf) showed how to make a neural net confuse Reese Witherspoon with Russell Crowe by adding a funky pair of technicolor zebra-striped wayfarer frames.  If that's all it takes, then maybe Clark Kent was onto something after all.

We had our own experience with this recently.  I trained a multi-class [DetectNet](https://github.com/NVIDIA/caffe/tree/caffe-0.15/examples/kitti) model to detect airplanes.  The model worked great, and it improved speed and accuracy over my previous models.  But when we ran tests on real images, we found that it didn't work very well when clouds were present.  The model usually produces confidence scores over 0.95 for clear images of big planes like airliners, but it was fooled by this image:

![A cloudy image]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/clouds_nodetections.png){: width="519px"}
![Detection problems]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/clouds_detections.png){: width="523px"}

To the human eye, it's very easy to pick out these airplanes because two of the three are only lightly obscured by clouds, and the third is only partially obscured.  But the DetectNet model is not used to dealing with clouds because the training imagery did not contain a lot of clouds.  As a result, it missed one plane altogether, it gave another plane a confidence value of 0.23, which is very low, and the third plane a confidence value of 0.90, which is still too low.  If you look closely at the plane that got the 0.23 score, you will see that it is lightly obscured by a cloud.  The other 0.90 plane is partially covered by a thin cloud, but you really have to zoom in to see it.  We can deal with this in the future by seeking out cloudy images to train our model with, but we can also work with the data we already have by adding synthetic clouds to clear images.

There are other kinds of variation in satellite imagery, like off-nadir angle, time of day, and atmospheric conditions.  We will need to address all of these in the future, so I'm expecting to do a lot more work on making models robust to image variations.

## Clouds in Satellite Imagery
For now, let's just deal with clouds.  I searched the internet for examples where people added clouds to images, and I mainly found instructions on [using graphics software](https://docs.gimp.org/en/python-fu-foggify.html) to [create clouds](http://smallbusiness.chron.com/create-perfect-clouds-gimp-37351.html), but I need a way to do this with code.  

One additional thing to consider before choosing a method is that most people are used to looking at clouds from ground level on planet earth, not from satellites in space.  So we should have a look at some clouds from satellite images and then decide how to proceed

![Satellite Clouds]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/31.jpg){: width="256px"}
![Satellite Clouds]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/12.jpg){: width="256px"}
![Satellite Clouds]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/45.png){: width="256px"}
![Satellite Clouds]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/desert_clouds1.jpg){: width="256px"}

As you can see, clouds actually look pretty much the same from space.  File that under "Today I Learned", and let's go make some clouds...

## Generating Clouds Programmatically
I found a great [example](http://lodev.org/cgtutor/randomnoise.html) creating clouds in C.  From looking at the source code, I can see that the author created some white noise, and then progressively upsampled smaller and smaller parts of the image to the original image size and stacked the results on top of each other.

Before I present my program in Python, here's the algorithm in plain English:

> 1. generate an NxN white noise image and store it
2. cut out the upper-left quadrant of the image
3. upsample the upper-left quadrant to the original image size, and store the result
4. repeat steps 2 and 3 until the corner image is 1x1
5. Sum the original noise pattern and all the upsampled quadrants pixel by pixel, scaling the values of each pattern by the inverse of the root of quandrant's edge dimension

I changed the order of the algorithm a little bit, but the result should be the same.

I'm going to reproduce this algorithm in Python.  I'm going to upgrade to bicubic interpolation because its 2017 and its a great time to be alive.

NOTE: This code was exported from a [Jupyter Notebook](https://github.com/DigitalGlobe/DeepCore/blob/master/assets/notebooks/clouds/Synthetic Clouds.ipynb) (python 3.4.3) [using nbconvert](http://briancaffey.github.io/2016/03/14/ipynb-with-jekyll.html).  There area a couple of things that are specific to Jupyter/iPython, like `%matplotlib inline` and the semicolon at the end.  If you want to run this code outside of a Jupyter Notebook, you will need to remove those.  Otherwise, it should work normally.


{% highlight python %}

import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import scipy
import scipy.ndimage
%matplotlib inline

def make_turbulence(im_size):
    # Initialize
    base_pattern = np.random.uniform(0,255, (im_size, im_size))
    turbulence_pattern = np.zeros((im_size, im_size))

    # Create cloud pattern
    power_range = range(2, int(np.log2(im_size)))
    for i in power_range:
        subimg_size = 2**i
        quadrant = base_pattern[:subimg_size, :subimg_size]
        upsampled_pattern = scipy.misc.imresize(quadrant, (im_size, im_size), interp='bicubic')
        # intperp can be 'nearest', 'lanczos', 'bilinear', 'bicubic' or 'cubic'
        turbulence_pattern += upsampled_pattern / subimg_size

    # Normalize values
    turbulence_pattern /= sum([1 / 2**i for i in power_range])
    
    return turbulence_pattern
    

turbulence_pattern = make_turbulence(768)

# Plot the results
_, ax = plt.subplots(figsize=(12, 12))
norm = matplotlib.colors.Normalize(vmin=0, vmax=255, clip=True)
ax.imshow(turbulence_pattern, cmap=plt.cm.gray, norm=norm)
ax.axis('off');

{% endhighlight %}

![Synthetic Clouds]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/Synthetic Clouds_0_0.png){: width="768px"}

We can make the clouds more granular by increasing the number 2 in `power_range = range(2, ...`.  I added the normalization step to make the code robust to that change.  It might be possible to save a few cycles by using the formula for the sum of a geometric series, but I'll leave that for another day.


## Adding Clouds to Images
Now that we can generate cloud patterns, lets add one to an image.  We want to leave the image more or less the same, but stick semi-transparent clouds on top of it.  In order to do that, we'll use PIL, the Python Image Library.  

First, let's load a totally clear, non-cloudy image.


{% highlight python %}
from PIL import Image

# Load the clear image
img = Image.open('61.png')

img
{% endhighlight %}


![png]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/Synthetic Clouds_2_0.png){: width="768px"}



Then, we'll convert the cloud pattern into a PIL format, so it's ready to be merged with another image.  Finally, we can merge the images together.


{% highlight python %}
# Convert the turbulence pattern into a PIL RGB image
turb_img = Image.fromarray(np.dstack([turbulence_pattern.astype(np.uint8)]*3))

# Make sure that the images match each other
print(img.mode, turb_img.mode)
print(img.size, turb_img.size)

# Now try blending them
Image.blend(img, turb_img, 0.5)
{% endhighlight %}

    RGB RGB
    (768, 768) (768, 768)





![png]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/Synthetic Clouds_4_1.png){: width="768px"}



We used the [`Image.blend`](http://pillow.readthedocs.io/en/3.4.x/reference/Image.html#PIL.Image.blend) function to combine the images.  The third parameter is the alpha value to use for blending.  If alpha is 0.0, we'll get the original image with no clouds.  If it's 1.0, we'll just get clouds with nothing else.  Let's try varying the alpha channel and see what it looks like.


{% highlight python %}
n_levels = 12
fig, axes = plt.subplots(3, n_levels//3, figsize=(20,16))
flag = True
for ax, alpha in zip(axes.flat, np.linspace(0,1,n_levels)):
    ax.imshow(Image.blend(img, turb_img, alpha))
    ax.axis('off')
    if flag:
        ax.set_title("alpha:    {:.2f}             ".format(alpha), fontsize=20, color='gray')
        flag = False
    else:
        ax.set_title("{:.2f}".format(alpha), fontsize=20, color='gray')
{% endhighlight %}


![png]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/Synthetic Clouds_6_0.png){: width="1000px"}


Now we can see a range of images with different alpha levels.  For augmentation, we'll want to avoid images that are so cloudy we can't make out the objects.  For this reason, we'll set an upper bound of 0.65 on the alpha parameter.  On the other hand, we don't want to waste our time creating augmented images that don't even look cloudy, so let's set a minimum alpha value of 0.2.

Now let's create an add_clouds function that we can use to augment images in the future.  To speed things up a little, we're dropping PIL from the augmentation process and just using pure numpy to add clouds to the image (I'm still using PIL to load the image).  This function assumes we're dealing with square images.  If you want it to work with non-square images, you'll have to modify the script to resize the clouds to match the image size.

Finally, let's add some clouds to [my favorite map projection](https://xkcd.com/977/).


{% highlight python %}
def add_clouds(image_np, alpha):
    turbulence_pattern = make_turbulence(image_np.shape[0])
    return (image_np*(1-alpha) + turbulence_pattern[:,:,None]*alpha).astype(np.uint8)

image_np = np.array(Image.open('Peirce_quincuncial_projection_SW_20W.JPG'))
alpha = np.random.uniform(0.2, 0.65)
cloudy_image_np = add_clouds(image_np, alpha)

_, ax = plt.subplots(figsize=(12,12))
ax.imshow(cloudy_image_np)
ax.axis('off');
{% endhighlight %}


![png]({{ site.baseurl }}/assets/images/Creating_Synthetic_Clouds/Synthetic Clouds_8_0.png){: width="768px"}

You can download my jupyter notebook [here]({{ site.baseurl }}/assets/notebooks/clouds/Synthetic Clouds.ipynb).  If you'd like to view it rendered, try [this link](https://github.com/DigitalGlobe/DeepCore/blob/master/assets/notebooks/clouds/Synthetic Clouds.ipynb).


