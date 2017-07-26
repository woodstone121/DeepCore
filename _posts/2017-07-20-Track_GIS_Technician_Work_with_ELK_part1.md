---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 1
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

With great training data come great results.

Object marking, otherwise known as digitizing, is when someone uses GIS software to outline and save ground features on a map for later analysis or use. These manually drawn polygons serve as georeferenced (stored with lat/lon location metadata) objects that can later be used as cookie cutters to isolate special features for analysis. For example, drawing squares (otherwise known as bounding boxes) around vehicles allows us to effectively "punch out" the satellite imagery at certain spots and perform machine learning on those patches of imagery. Digitize thousands of vehicles, and you have a solid dataset that when fed into a machine learning model, can automatically detect vehicles in satellite imagery.

![Digitizing example]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Digitizing_Example.png){: width="45%"} ![Detection image]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Detection_Image.png){: width="45%"}

A digitizer creates bounding boxes for vehicles in GIS software (left), and vehicle detection results after training data is fed into neural network (right)

When we are trying to do multi-object detection, quality training data is especially necessary due to the nuances in object types. We're trying to detect cars. Buses. Buildings. Planes. Hundreds of other things. It’s important that we not only collect enough training data for each object type to feed our neural networks, but also in a variety of geographic locations, in equal densities per object type, among other quality control factors.

As part of the work-flow, it's important that digitizers are keeping a consistent pace, maintaining an even distribution of work amongst themselves, collecting the right amount of object types in the right geographic locations, and otherwise hitting important benchmarks each day/week/month of the project duration to assure quality digitizing. With any task that involves high level of repetition, digitizers could get bored and/or distracted which can affect the quality of their work. So, providing a visual interface that breaks down rate, quantity, and other metrics per digitizer creates an incentive for technicians to provide quality work, as the details of their hourly work is reported on the website for managers and other digitizers to view. It can also serve as a leaderboard for the digitizers, where technicians can compete to mark the most objects or have the best hourly rate. If you work in a geospatial setting and oversee a group of digitizers, setting up a dashboard using the [ELK](https://www.elastic.co/webinars/introduction-elk-stack) ([Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html), [Logstash](https://www.elastic.co/guide/en/logstash/current/index.html), [Kibana](https://www.elastic.co/guide/en/kibana/5.5/index.html)) stack to track work progress could be beneficial to your team. Regardless of your specific application, the Kibana visualizations provide a comprehensive breakdown of your project from start to finish, which is always good information to have when anticipating deadlines. It’s definitely helped us in our own projects, and provided a layer of transparency between us and our employees and our training data.

We've stood up a really nice looking website dashboard to display metrics about the data our digitizers are gathering and the quality of their individual and collective work. In this series, I'll give you an overview of how it's done.

![Kibana dashboard example]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Kibana_dashboard_example.png){: width="100%"}
Our dashboard, which updates in near real-time. Visualizations include total number of objects digitized today (row 1, column 1), total count since start of project with goal (row 2, column 2), cumulative counts per day per employee (row 2, column 2), and heat map showing location of digitized polygon densities (row 5, column 1).

Here’s what you’ll need to get started:
1.  GIS software program (we use [QGIS](http://www.qgis.org/en/site/forusers/download.html)) for digitizing
2. [Postgres](https://www.postgresql.org/download/) installation on a server such as an [Amazon EC2 instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html)
3. ELK stack installed on the server

[Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html), [Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html), and [Kibana]() (ELK) are three separate open-source components that can work together in this stack to serve as a powerful analytics platform. Elasticsearch(ES) provides a NoSQL database and the ability to do full text searches through an HTTP interface, Logstash provides the gathering, filtering, and storing of data into ES (essentially the I/O component), and Kibana provides the visualization layer.

In the next few posts, I’ll describe the project pipeline from digitizing in QGIS to displaying the endlessly cool visualizations that Kibana is capable of projecting with your data. This is Part 1 of that series.

# Part 1: Digitize Polygons in QGIS and Store them in PostgresDB
First, install [Postgres](https://www.postgresql.org/download/) on your server machine. I also recommend installing [pgAdmin3](https://www.pgadmin.org/download/) so that you can easily query and view your database with a GUI. You'll need to install the [PostGIS extension](http://postgis.net/install/) so that you can work with geospatial data types and functions in Postgres.

Within your database, you'll need to create a table where you'll store the digitized objects. You must add a field for the digitization, it would be of type `geometry`, which is a special built-in [geospatial datatype](https://postgis.net/docs/using_postgis_dbmanagement.html) within Postgres that can be used with the PostGIS extension enabled. Add other fields that you think would be valuable metadata, such as a `feature_id`, `type_id`, `author`, etc.

Create user accounts for all your digitizers on your Postgres database and grant them access to the table.

Now, each one of the users should be able to go into QGIS (or other GIS software of your choice) and connect to the Postgres database with their credentials, draw a polygon, and have a row of data added. In our database, we set it up so that each digitization populates the `ingest_time` to the current time, the `edited_by` to the current user's, and the `axis_bbox` to be the geometry of the digitized polygon. The user will (obviously) need to key in the `type_id` value. Additionally, through the use of Postgres [triggers](https://www.postgresql.org/docs/9.1/static/sql-createtrigger.html), each time a user draws a polygon, the `point_geom` column is populated with the `axis_bbox` centroid.

Now for the actual digitizing. In QGIS, we can to our Postgres table and add it as a layer. Since we'll want near real time updates, make sure your digitizers are saving their edits as frequently as possible.

![Postgres button in QGIS]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/QGIS_postgres_connect.png){: width="45%"} ![Connect Postgres]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Connect_Postgres.png){: width="45%"}

In our example digitization here, we are giving the attribute `type_id` a value of `1`. As stated earlier, we also set up our database so that `edited_by`, `feature_id`, `ingest_time`, and `point_geom` are automatically propagated. Depending on how you set up your table in Postgres, you'll be able to enter in whatever attributes you need after each digitization. Saving edits and querying the database afterwards shows that your new digitization and all its metadata has been stored in the database.

![Select Object]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Select_Object.png){: width="85%"}

![Finish Polygon]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Finish_Polygon.png){: width="85%"}

![Key in Object Type]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Key_In_Object_Type.png){: width="85%"}

We see our digitization is `edited_by` by user `tom`, our keyed in `type_id` of value `1`, the digitization itself stored in `axis_bbox`, and the centroid calculated by the Postgres trigger in `point_geom`.

![Query Result]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/query_result.png){: width="85%"}

Summary
=======
We talked about the value of setting up a dashboard to monitor digitizing projects, creating a Postgres database and table on a server, and enabling the capability to digitize in QGIS while saving edits to the Postgres database.

Next in Part 2, we'll examine how to transfer the data from our PostgresDB into the Elasticsearch framework.

![alt-text]({{ site.baseurl }}/images/banner2.png)
