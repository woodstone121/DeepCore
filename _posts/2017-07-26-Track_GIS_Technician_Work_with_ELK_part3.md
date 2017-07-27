---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 3
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

Part 3: Use Kibana to Visualize Data from Elasticsearch
=======

In this final part of the series, I'll be showing you visualizations that can be done with Kibana using the data in Elasticsearch. This is the fun part, and there's no coding required!

*Note: There are countless visualizations you can make in Kibana, but these were a few of our favorites for the purpose of monitoring a digitizing project*

One interesting and useful visualization for looking at digitized objects is the Coordinate Map. It aggregates our location data from Elasticsearch into a map of your choice (here we've shown a graduated symbol map), where the size and color of the circles refer to the density of objects marked in a region.

![Graduated Symbol Map]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Graduated_Symbol_Map.png){: width="100%"}
*Hovering over each map symbol shows the number of objects digitized in a geographic region*

This could be helpful if, for example, you intend to detect forest cover, but your training data is located over deserts (that would not yield accurate results at all). In this case, you'd be able to tell right away from the visualization (which keep in mind, is updating every minute, or however long you've specified in Logstash) and notify your digitizers to divert their efforts appropriately, saving time and resources. On the other hand, if you want your object markings to be dispersed heterogeneously around the globe, the visualization would be able to confirm this result as well.

Here's another scenario. We're curious about how productive our digitizers are being on an hourly basis. Kibana is able to aggregate our ES data by timestamp attribute we populated our features with. This gives us an hourly time series graph with the number of objects digitized every hour of the workday since our project began a few weeks ago:

![Object Counts Hourly]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Object_Counts_Hourly.png){: width="100%"}

From this visualization, we can see where the peaks and valleys of the work day are, and how many objects were counted for each hour and of each day of the project. We can see which days and hours are more productive than others.

We can aggregate our Elasticsearch data by count and object type as well. This bar graph shows the total counts of each of the objects:

![Object Type Counts]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Object_Type_Counts.png){: width="100%"}

We can aggregate by count and person. This one shows each bar as an editor and the number of objects counted by that editor. *Note: names changed to protect identity of our digitizers*

![Object Counts per User]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Object_Counts_per_User.png){: width="100%"}


We can even create a split series, where we get a breakdown of which objects comprise each users' total count:

![User Object Breakdown]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/User_Object_Breakdown.png){: width="100%"}

In this line graph, we've created a split series where each line is a digitizer. The x-axis referes to the date and the y-axis refers to the counted objects. These are the cumulative object counts since the beginning on the project. We can see which users have been collecting the most objects, and which users are working at the fastest rates.

![Cumulative Counts by User]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Cumulative_Counts_by_User.png){: width="100%"}

We can use the Metric and Goal visualizations to display important numbers too. We can display today's total number of digitized objects as well as the total number of digitized objects since the beginning of the project.

![Today Total Count]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Today_Total_Count.png){: width="100%"}
*15,568 features digitized today using the Metric visualization (left), 433,330 objects digitized in total using the Goal visualization (right). The Goal visualization also visualizes the fraction done in comparison to a goal number of digitized objects by the end of the project*

Here are all the examples we just discussed, together in a dashboard:

![Dashboard Example]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/dashboard_example.png){: width="100%"}

Summary
=======
In this last section, we talked about a few visualizations that are possible using Kibana. A dashboard like this one will allow you to monitor all kinds of metrics easily and quickly, such as digitizer progress, hourly rate, total counts, object type counts, and geospatial distribution.

![alt-text]({{ site.baseurl }}/images/banner2.png)
