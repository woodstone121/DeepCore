---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 1
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

With great training data come great results.

When the subject is multi-object detection, this is especially true due to the subtle nuances in object types. We're trying to detect cars. And buses. And buildings. And planes. And hundreds of other things through satellite imagery using machine learning algorithms. It’s important that we not only collect enough training data for each object type to feed our neural networks, but also in a variety of geographic locations, in equal densities per object type, among other quality control factors.

We also want to make sure our GIS technicians doing the object marking are keeping a consistent pace, maintaining an even distribution of work amongst themselves, and otherwise hitting important benchmarks each day/week/month of the project duration to assure quality digitizing. Similarly, if you work in a geospatial setting and oversee a group of digitizers, setting up a dashboard using the [ELK](https://www.elastic.co/webinars/introduction-elk-stack) ([Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html), [Logstash](https://www.elastic.co/guide/en/logstash/current/index.html), [Kibana](https://www.elastic.co/guide/en/kibana/5.5/index.html)) stack to track work progress could be beneficial to your team. I know it’s definitely helped us, and provided a layer of transparency between us and our employees and our training data.

[We've stood up a really nice looking website dashboard](http://deepcore.io/maas) to show all sorts of metrics about the data our employees are gathering and the quality of their individual and collective work. I'll give you an overview of how it's done. Read on to try it for yourself!

![Kibana dashboard example]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Kibana_dashboard_example.png){: width="100%"}
Our dashboard, which updates near real-time. Visualizations include total number of objects digitized today (row 1, column 1), total count since start of project with goal (row 2, column 2), cumulative counts per day per employee (row 2, column 2), and heat map showing location of digitized polygon densities (row 5, column 1).

Here’s what you’ll need to get started: a GIS software program (we use [QGIS](http://www.qgis.org/en/site/forusers/download.html)) for digitizing, a [Postgres](https://www.postgresql.org/download/) installation on a server such as an [Amazon EC2 instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html), and the ELK stack installed on the server. [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html), [Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html), and [Kibana]() (ELK) are three separate open-source components that can work together in this stack to serve as a powerful analytics platform. Elasticsearch(ES) provides a NoSQL database and the ability to do full text searches through an HTTP interface, Logstash provides the gathering, filtering, and storing of data into ES (essentially the I/O component), and Kibana provides the visualization layer.

In the next few posts, I’ll describe the entire work-flow from 1) drawing polygons in QGIS and storing them in PostgresDB in real-time to 2) collecting, manipulating, and ingesting the data into Elasticsearch via Logstash, and then 3) displaying the visualizations on a web page with Kibana. This is Part 1 of that series.

# Part 1: Store Digitized Polygons from QGIS into PostgresDB Table
First, install [Postgres](https://www.postgresql.org/download/) on your server machine (we'll use localhost for this example but you'll need a live server for production purposes). I also recommend installing [pgAdmin3](https://www.pgadmin.org/download/) so that you can easily query and view your database with a GUI. You'll need to install the [PostGIS extension](http://postgis.net/install/) so that you can work with geospatial data types and functions in Postgres. Next, we'll create database `postgres_db` and enable the PostGIS extension. In psql or pgadmin3 execute:

```
CREATEDB postgres_db;
CREATE EXTENSION postgis;
```


 Within our database `postgres_db` we'll create table `objects_table`. We'll need the following columns: `feature_id` for a unique identifier, `edited_by` to show who digitized it, `type_id` to reflect what kind of object it represents, `ingest_time` to show when it was last edited, and `axis_bbox` to store the digitized polygon (in this example we will use bounding boxes), and `point_geom` to store the bounding box centroid. In psql or pgadmin3, execute:

```
CREATE TABLE public.objects_table (
	feature_id SERIAL,
	edited_by text DEFAULT "current_user"(),
	ingest_time timestamp with time zone DEFAULT now(),
	type_id integer NOT NULL,
	axis_bbox geometry(MultiPolygon, 4326),
	point_geom geometry(Point, 4326),
	CONSTRAINT objects_table_pkey PRIMARY KEY (feature_id)
)
WITH (
	OIDS=TRUE
);
```

Since we'll want near real-time updates, the Postgres database table will be added as a layer in QGIS and digitizers will add features to the table directly in an edit session (as opposed to creating shapefiles, then dumping it all into the PostgresDB at the end of the day). Make sure digitizers save edits often so that the database is always as current as possible.


Create users and passwords for all of your digitizers and grant them access to the table (select, insert, delete are good starters). Now, each one of the users should be able to go into QGIS and connect to the Postgres database with their credentials. We're not done yet though - each time the digitizers draw a polygon, we'll need to propagate the `ingest_time`, `geom`, and `employee_name` fields. To do this, add triggers to the database columns. You'll need to default the `ingest_time field` to the current time on an INSERT or SELECT and the `edited_by` field to the digitizer's username. QGIS will automatically populate the `geom` column when you insert a feature.

```
CREATE USER tom WITH PASSWORD 'password';
CREATE USER john WITH PASSWORD 'password';
CREATE USER jane WITH PASSWORD 'password';
CREATE USER sarah WITH PASSWORD 'password';
```

Then, grant permissions to your users for the table and the sequence
```
GRANT ALL ON TABLE public.objects_table TO tom, john, jane, sarah;
GRANT ALL ON SEQUENCE public.objects_table_feature_id_seq TO tom, john, jane, sarah;
```

One last Postgres configuration: we'll want to create a trigger. This trigger will fire after each object is digitized, find the centroid of the polygon, and populate the centroid_geom column with this value. The centroid is needed in Part 3, when we want to visualize our objects as points.

Create the trigger:

```
CREATE OR REPLACE FUNCTION public.objects_table_enhance_data_trigger() ON public.objects_table
  RETURNS trigger AS
$BODY$
	DECLARE
		point_geom geometry;
        BEGIN
		SELECT INTO point_geom ST_CENTROID(NEW.axis_bbox);
		NEW.point_geom = point_geom;

                RETURN NEW;
        END;

$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100;
```

Grant permissions to the trigger:
```
ALTER FUNCTION public.objects_table_enhance_data_trigger()
  OWNER TO postgres;
GRANT EXECUTE ON FUNCTION public.objects_table_enhance_data_trigger() TO public;
```

Apply the triggger to the table:
```
CREATE TRIGGER objects_table_enhance_data_trigger
  BEFORE INSERT
  ON public.objects_table
  FOR EACH ROW
  EXECUTE PROCEDURE public.objects_table_enhance_data_trigger();

```

Okay, now let's see all our hard work in action. Let's get to digitizing! Open up QGIS and connect to your PostgresDB and table

![Postgres button in QGIS]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/QGIS_postgres_connect.png){: width="45%"} ![Connect Postgres]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Connect_Postgres.png){: width="45%"}

Edit the postgres table layer by creating a polygon and keying in the `type_id` field. Save your edits. Querying the database immediately afterwards shows that your new digitization is present.

![Connect Postgres]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/query_result.png){: width="85%"}

We see our digitization is `edited_by` by user `tom`, our keyed in `type_id` of value `1`, the digitization itself stored in `axis_bbox`, and the centroid calculated by the Postgres trigger in `point_geom`. The `feature_id` here shows `7` because I tested a few rows before this example, but as your first entry, you should get a `feature_id` of `1`.

Summary
=======
Congratulations! You've successfully set up a Postgres database and table on your server, added users, granted permissions, created trigger functions, and enabled the capability to monitor your data as your digitizers work in real time! Feel free to add more sophisticated fields, triggers, and user groups per your requirements.

Next in Part 2, we'll examine how to transfer the data from our PostgresDB into the Elasticsearch framework.

![alt-text]({{ site.baseurl }}/images/banner2.png)
