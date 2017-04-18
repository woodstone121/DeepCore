---
title: Working With Large Tiff Files
author: Alan J. Schoen
published: False
---

At DG, we we produce large, high-quality images.  So when I pull images from GBDX, they are often very big, even up to 30GB.  Working with these images has been a challenge, so I thought I'd share my best practices for working with these images.

Usually, when I'm working with a big tiff file, I'm really only interested in a small portion of that file.  For example, I might have a XXXX x XXXX strip, but I'm only interested in a XXX x XXX area representing an airport.  So I can just pull out the area of the image that represents the airport, and work with that.  Fortunately, GDAL can read in just that portion of the image without reading the entire thing into memory.

SHOW IMAGE

My team uses [GDAL](http://www.gdal.org/) to read and write geospatial data.  It's a big open-source project, and it lets us read and write many different file formats, but it can be a challenge to work with sometimes.  The interface is complicated, and the documentation is sometimes incomplete.  So it took me some time to figure out the simplest way to pull out a region from a tiff file.

### Installing the right version of GDAL
We need to get one thing out of the way first: if you're working with a gigantic tiff file, you're going to need a recent version of GDAL.  GDAL uses libtiff to read tiff files, and [you need a recent version](http://www.gdal.org/frmt_gtiff.html) to read tiff files larger than 4GB (libtiff >= 4.0).

Installing an up-to-date version of GDAL on Ubuntu is easy, using the ubuntugis ppa:
```
sudo add-apt-repository ppa:ubuntugis/ppa
sudo apt-get update
sudo apt-get install libgdal-dev
sudo apt-get install gdal-bin
```

If you're on another platform, it can be much harder.

### Loading part of a tiff using gdal_translate

The gdal_translate function is designed to convert between different kinds of files.  In this case, we're converting a large file into a small file.  Fortunately, gdal_translate has pretty good [documentation](http://www.gdal.org/gdal_translate.html).

```
```

### Converting to a UTM zone
I prefer UTM-projected data for machine learning.  UTM is great because it lets us treat geospatial images like normal images.  The resolution units are in meters, and the pixels are always square, so we don't have to worry about objects getting distorted with changes in lattitude.  Unfortunately, UTM zones are a little complicated because, unlike WGS84, UTM isn't just a big projection that includes the entire world.  UTM divides the world into zones, and you need to make sure that you're working in the right zone.

show UTM map

The National Geospatial Intelligence Agency has a [great page describing UTM](http://earth-info.nga.mil/GandG/coordsys/grids/universal_grid_system.html) which includes a [link to a shapefile](http://earth-info.nga.mil/GandG/coordsys/zip/UTM/zip/UTM_Zone_Boundaries/UTM_Zone_Boundaries.zip) dividing the world into UTM zones.  We can use [that shapefile](http://earth-info.nga.mil/GandG/coordsys/zip/UTM/zip/UTM_Zone_Boundaries/UTM_Zone_Boundaries.zip) to find the right zone to work in.

I'll assume we're starting with WGS84 projected data, which we will need to convert to UTM.