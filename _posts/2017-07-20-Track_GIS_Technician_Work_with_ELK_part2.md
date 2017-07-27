---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 2
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

In my last post, I introduced the concept of monitoring a digitizing project using the ELK stack. We began by storing our digitized polygons drawin in QGIS into a database. This is Part 2, where I'll describe how we sync the data from our database to Elasticsearch so we can actually utilize the Elastic stack.

Part 2: Use Logstash to Sync Data from Database to Elasticsearch
=======

Logstash is the I/O layer of the Elk stack. All it does is take in an input and output data to Elasticsearch. Since our input is from a database, make sure you have the correct JDBC driver installed.

The skeleton of your logstash config file will look something like this:

```
input {
    jdbc {
        jdbc_connection_string => "your_connection_string_here"
        jdbc_user => "your_username_here"
        jdbc_driver_library => "your_java_classpath_here"
        jdbc_driver_class => "name_of_driver_class"
        statement => "your_query_here"
    }
}


output {
    elasticsearch {
        index => "your_index_here"
        document_type => "your_type_here"
        hosts => "localhost:9200"

```

As you can see, the input section contains [different parameters](https://www.elastic.co/guide/en/logstash/5.3/plugins-inputs-jdbc.html) such as credentials for logging into the database, SQL statement, and you can even add a scheduler for how often the SQL statement should be run.

If you want to be extra fancy, you can add a filter between the input and output which allows you to change up the data retrieved from the SQL statement before it's output to Elasticsearch.


Summary
=======
In this section, we explained how Logstash is the intermediary between your database and your Elasticsearch index. Logstash is important for syncing your data in near real time with the Elastic stack.

In the final (and most fun) section, we'll look at how Kibana can take the data in Elasticsearch and visualize the data in really neat ways.

![alt-text]({{ site.baseurl }}/images/banner2.png)
