---
layout: post
title: Track GIS Technician Work with the ELK Stack
author: Gina Li
---

Great training data produce great results. 

When it comes to multi-object detection, this is especially true due to the subtle nuances in object types. Say we want to be able to detect cars. And buses. And buildings. And planes. And many other things. It’s important that we not only collect enough training data for each object type, but also in a variety of geographic locations. 

We also want to make sure the GIS technicians who are marking these objects are keeping a consistent pace, maintaining an even distribution of work amongst themselves, and otherwise hitting important benchmarks each day to assure good quality digitizing. 

I’m going to describe to you how we were able to create a really nice looking website dashboard to show all sorts of metrics about the data our employees collected, and the quality of their individual work. Similarly, if you work in a geospatial setting and you oversee many digitizers, setting up a dashboard using the ELK stack to track details of work being done would be beneficial to you team. I know it’s definitely helped us, and provided a layer of transparency between us and our technicians.

Elasticsearch, Logstash, and Kibana (ELK) are three separate open-source components that work together to serve as an analytics platform. In other words, say we have a lot of data (hundreds of thousands of georeferenced polygons). Elasticsearch provides a NoSQL database and the ability to do full text searches through an HTTP interface, Logstash provides the gathering, filtering, and storing of logs, and Kibana provides the visualization layer.


![alt-text]({{ site.baseurl }}/images/banner2.png)
