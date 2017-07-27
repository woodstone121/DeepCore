---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 1
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

With great training data come great results.

Object marking, otherwise known as digitizing, is when someone uses GIS software to outline and save ground features on a map. These manually drawn polygons serve as georeferenced (stored with lat/lon location metadata) objects that can be used as cookie cutters to isolate special features for analysis. For example, drawing squares (otherwise known as bounding boxes) around vehicles allows us to effectively "punch out" the satellite imagery at certain spots and perform machine learning on those pixels of imagery. Digitize thousands of vehicles, and you have a solid dataset that when fed into a neural network, can automatically detect vehicles in satellite imagery.

![Digitizing example]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Digitizing_Example.png){: width="45%"} ![Detection image]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Detection_Image.png){: width="45%"}

A digitizer creates bounding boxes for vehicles in GIS software (left), and vehicle detection results after training data is fed into neural network (right)

When we are trying to do multi-object detection, quality training data is especially necessary due to the nuances in object types.

On the data side, we're trying to detect cars. Buses. Buildings. Planes. Hundreds of other things. It’s important that we not only collect enough training data for each object type to feed our neural networks, but also in a variety of geographic locations, in equal densities per object type, among other quality control factors.

On the human side, we want to make sure our digitizers are working at a consistent pace, distributing work fairly amongst themselves, collecting the right amount of object types, and otherwise hitting important benchmarks. With any task that involves high level of repetition, digitizers could get bored and/or distracted which can affect the quality of their work. So, providing a visual interface that breaks down rate, quantity, and other metrics per digitizer incentivizes technicians to provide quality work, as the details of their hourly work is reported on the website for managers and other digitizers to view. It can also serve as a leaderboard for the digitizers, where technicians can compete to mark the most objects or have the best hourly rate. If you work in a geospatial setting and oversee a group of digitizers, setting up a dashboard using the [ELK](https://www.elastic.co/webinars/introduction-elk-stack) ([Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html), [Logstash](https://www.elastic.co/guide/en/logstash/current/index.html), [Kibana](https://www.elastic.co/guide/en/kibana/5.5/index.html)) stack to track work progress could be beneficial to your team.

Regardless of your specific application, the Kibana visualizations provide a comprehensive breakdown of your project from start to finish, which is always good information to have when anticipating deadlines. It’s definitely helped us in our own projects, and provided a layer of transparency between us and our employees and our training data.

We've stood up a really nice looking website dashboard to display metrics about the data our digitizers are gathering and the quality of their individual and collective work. In this series, I'll give you an overview of how it's done.

![Kibana dashboard example]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Kibana_dashboard_example.png){: width="100%"}
*Our dashboard, which updates in near real-time. Visualizations include total number of objects digitized today (row 1, column 1), total count since start of project with goal (row 2, column 2), cumulative counts per day per employee (row 2, column 2), and heat map showing location of digitized polygon densities (row 5, column 1).*

Here’s what you’ll need to get started:
1. GIS software program (we use [QGIS](http://www.qgis.org/en/site/forusers/download.html)) for digitizing
2. A database with geospatial capabilities such as [Postgres](https://www.postgresql.org/download/) with the [PostGIS extension](http://postgis.net/install/)
3. A server instance such as an [Amazon EC2 instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html)
4. ELK stack installed on the server

[Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html), [Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html), and [Kibana]() (ELK) are three separate open-source components that can work together in this stack to serve as a powerful analytics platform. Elasticsearch(ES) provides a NoSQL database and the ability to do full text searches through an HTTP interface, Logstash provides the gathering, filtering, and storing of data into ES (essentially the I/O component), and Kibana provides the visualization layer.

In the next few posts, I’ll describe the project pipeline from digitizing in QGIS to displaying the endlessly cool visualizations that Kibana is capable of projecting with your data. This is Part 1 of that series.

# Part 1: Digitize Polygons in QGIS and Store them in Database
In a database table, add fields to record data you'd like to display in your dashboard, like time of entry, who did the entry, and the geometry.

Within the GIS software, we can connect to our database table and begin adding features.

Depending on how you set up your table in Postgres, you'll be able to enter whatever attributes you need after each digitization. In our application, we've included an object type id, who edited it, time of entry, and the centroid.

![Select Object]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Select_Object.png){: width="85%"}

![Key in Object Type]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Key_In_Object_Type.png){: width="85%"}

![Finish Polygon]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Finish_Polygon.png){: width="85%"}

Saving edits and querying the database afterwards shows that your new digitization and all its metadata has been stored in the database.

![Query Result]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/query_result.png){: width="85%"}

Summary
=======
We talked about the value of setting up a dashboard to monitor digitizing projects, creating a database and table on a server, and enabling the capability to digitize in GIS software while saving edits in near real time to the database.

Next in Part 2, we'll examine how to transfer the data from our PostgresDB into the Elasticsearch framework.

![alt-text]({{ site.baseurl }}/images/banner2.png)
