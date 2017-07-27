---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 3
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

Part 3: Use Kibana to Visualize Data from Elasticsearch
=======

In this final part of the series, we will be constructing visualizations with Kibana using the data in Elasticsearch. This is the fun part, and there's no coding required!

One interesting and useful visualization for digitizing projects is the Coordinate Map. It aggregates our location data from Elasticsearch into a map of your choice (here we've shown a graduated symbol map), where the size and color of the circles refer to the density of objects marked in a region.

![Graduated Symbol Map]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Graduated_Symbol_Map.png){: width="95%"}
*Hovering cursor over each map symbol shows the number of digitizations in a geographic region*

This could be helpful if for example, you intend to detect forest cover, but your training data is located over deserts (that would not yield accurate results at all). In this case, you'd be able to tell right away from the visualization (which keep in mind, is updating every minute) and notify your digitizers to divert their efforts and effectively save valuable time and resources. On the other hand, if you want your object markings to be dispersed all over a variety of different locations, the graduated circle would be able to verify these results immediately while the digitizing is happening.

Here's another scenario. We're curious about how productive our digitizers are being on an hourly basis. Kibana is able to aggregate our ES data by timestamp attribute we populated our features with. This gives us an hourly time series graph with the number of objects digitized every hour of the workday since our project began a few weeks ago:

![Object Counts Hourly]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Object_Counts_Hourly.png){: width="85%"}
