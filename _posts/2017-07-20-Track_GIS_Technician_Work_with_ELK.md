---
layout: post
title: Track GIS Technician Work with the ELK Stack
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

With great training data come great results.

When the subject is multi-object detection, this is especially true due to the subtle nuances in object types. Say we want to be able to detect cars. And buses. And buildings. And planes. And many other things through satellite imagery. It’s important that we not only collect enough training data for each object type to feed our models, but also in a variety of geographic locations, among other quality control factors.

We also want to make sure our GIS technicians doing the object marking are keeping a consistent pace, maintaining an even distribution of work amongst themselves, and otherwise hitting important benchmarks each day to assure quality digitizing. Similarly, if you work in a geospatial setting and you oversee many digitizers, setting up a dashboard using the [ELK](https://www.elastic.co/webinars/introduction-elk-stack) ([Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html), [Logstash](https://www.elastic.co/guide/en/logstash/current/index.html), [Kibana](https://www.elastic.co/guide/en/kibana/5.5/index.html)) stack to track work progress could be beneficial to your team. I know it’s definitely helped us, and provided a layer of transparency between us and our employees and our training data.

We were able to create a really nice looking website dashboard to show all sorts of metrics about the data our employees collected and the quality of their individual and collective work. Read on to try it for yourself!

![Kibana dashboard example][Kibana_dashboard_example]
######Kibana is able to create all sorts of customizable visualization from the data we’ve provided in Elasticsearch. Time-series visualizations are especially helpful for managers to log daily/hourly/weekly progress as the project goes on.

Here’s what you’ll need: a GIS software program (we used [QGIS](http://www.qgis.org/en/site/forusers/download.html)) for digitizing, a [Postgres](https://www.postgresql.org/download/) installed on a server such as an [Amazon EC2 instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html), and the ELK stack installed on the server. [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html), [Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html), and [Kibana]() (ELK) are three separate open-source components that can work together in this stack to serve as a powerful analytics platform. Elasticsearch provides a NoSQL database and the ability to do full text searches through an HTTP interface, Logstash provides the gathering, filtering, and storing of data into ES (essentially the I/O component), and Kibana provides the visualization layer. I also recommend installing [pgAdmin3](https://www.pgadmin.org/download/) so that you can easily view and query and visualize your database with a GUI.

In the next few posts, I’ll describe the entire work-flow from drawing polygons in QGIS to collecting, manipulating, and ingesting the data into Elasticsearch via Logstash, and then displaying the visualizations on a web page with Kibana. This is Part 1 of that series.

# Part 1: Store Digitized Polygons in PostgresDB Table
First, install [Postgres](https://www.postgresql.org/download/) on your machine. Create a database and a table. In this example, we've created database `postgres_db` and table `digitized_polygons`. For the `digitized_polygons` table, we'll want the following columns: feature_id (integer) as the auto incremented primary key, employee_name (text), type_id(integer), ingest_time(timestamp with time zone), and geom (geometry)

(insert visual here)

Create users and passwords for all of your digitizers and grant them access to the table (select, insert, delete are good starters). Now, each one of the users should be able to go into QGIS and connect to the Postgres database with their credentials. We're not done yet though - each time the digitizers draw a polygon, we'll need to propagate the `ingest_time`, `geom`, and `employee_name` fields. To do this, add three triggers to the database.

![Connect to postgres through QGIS][QGIS_postgres_connect]

###### default the `ingest_time` to the current timestamp, `geom` to the feature created, and `employee_name` to the current username.

Ideally, for near real-time updates, add your Postgres database table as a layer in QGIS and add features to your table directly in an edit session(as opposed to creating shapefiles, then dumping it into the PostgresDB). Make sure digitizers save edits often so that the database is always as current as possible.

Congratulations, you've successfully created a database, added users, granted permissions, created trigger functions, and enabled the capability to monitor your database as your digitizers work in real time! Feel free to add more sophisticated fields, triggers, and user groups per your requirements.

Next, we'll examine how to transfer the data from our PostgresDB into the Elasticsearch framework.

![alt-text]({{ site.baseurl }}/images/banner2.png)

[QGIS_postgres_connect]: {{site.baseurl}}/assets/images/2017-07-20-Track_GIS_Technicion_Work_with_ELK/QGIS_postgres_connect.png “Connect to postgres through QGIS” {:width=“80%”}
