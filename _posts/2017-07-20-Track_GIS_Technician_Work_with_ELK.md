---
layout: post
title: Track GIS Technician Work with the ELK Stack
author: Gina Li
desc: How we tracked employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

With great training data come great results.

When it comes to multi-object detection, this is especially true due to the subtle nuances in object types. Say we want to be able to detect cars. And buses. And buildings. And planes. And many other things. It’s important that we not only collect enough training data for each object type to feed our models, but also in a variety of geographic locations.

We also want to make sure the GIS technicians who are marking these objects for us are keeping a consistent pace, maintaining an even distribution of work amongst themselves, and otherwise hitting important benchmarks each day to assure quality digitizing. Similarly, if you work in a geospatial setting and you oversee many digitizers, setting up a dashboard using the ELK (Elasticsearch, Logstash, Kibana) stack to track details of work being done could be beneficial to you team. I know it’s definitely helped us, and provided a layer of transparency between us and our employees and data.

We were able to create a really nice looking website dashboard to show all sorts of metrics about the data our employees collected and the quality of their individual and collective work. Read on to try it for yourself!

(insert picture here)
Kibana is able to create all sorts of customizable visualization from the data we’ve provided in Elasticsearch. Time-series visualizations are especially helpful for managers to log daily/hourly/weekly progress as the project goes on.

Here’s what you’ll need: a GIS software program (we used QGIS) for digitizing, a PostgresDB and table set up on a server (Amazon’s EC2 instance works well), and the ELK stack installed on the server. Elasticsearch, Logstash, and Kibana (ELK) are three separate open-source components that can work together to serve as a powerful analytics platform. Elasticsearch provides a NoSQL database and the ability to do full text searches through an HTTP interface, Logstash provides the gathering, filtering, and storing of data into ES (essentially the I/O component), and Kibana provides the visualization layer.

In the next few sections, I’ll describe the entire work-flow from drawing polygons in QGIS to collecting, manipulating, and ingesting the data into Elasticsearch via Logstash, and displaying the visualizations on a web page with Kibana.

# Part 1: Store Digitized Polygons in PostgresDB Table
Ideally, for near real-time updates, add your Postgres database table as a layer in QGIS and add features to it directly in an edit session(as opposed to creating shapefiles, then dumping it into the PostgresDB). Make sure to save often.


![alt-text]({{ site.baseurl }}/images/banner2.png)
