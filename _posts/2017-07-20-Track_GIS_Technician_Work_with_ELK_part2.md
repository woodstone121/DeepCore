---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 2
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

In my last post, I introduced the concept of monitoring a digitizing project using the ELK stack. I showed you some of our current dashboard visualizations, talked about the level of detail that ELK can provide for GIS technicians and their managers, and explained how the Postgres database propagates our digitized data in near real-time. This is Part 2, where I'll describe how we sync the data from Postgres to Elasticsearch so we can actually utilize the Elastic stack.

Part 2: Use Logstash to Sync Data from PostgresDB to Elasticsearch
=======

First, we'll want to install [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html) and [Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html) if you haven't already. Also install the [Postgres JDBC driver](https://jdbc.postgresql.org/download.html) which allows you to read input from Postgres and output results to Elasticsearch.

Logstash is the I/O layer of the Elk stack. Our input is the Postgres database and table and the output is the Elasticsearch index and type. A filter can also be used, which munges the input Postgres data before it is output to Elasticsearch.

To run properly, the logstash directory (usually located in `/usr/share/logstash`, but see [the official guide](https://www.elastic.co/guide/en/logstash/5.4/dir-layout.html)) will need a `.conf` file. The file would look something like this for our applications:

```
input {
    jdbc {
        # Postgres jdbc connection string to postgres_db database
        jdbc_connection_string => "jdbc:postgresql://localhost:5432/postgres_db"
        # The user we wish to execute our statement as
        jdbc_user => "ginal"
        jdbc_password => "password"
        # The path to our downloaded jdbc driver
        jdbc_driver_library => "/usr/lib/jvm/java-8-oracle/lib/postgresql-42.1.1.jre6.jar"
        # The name of the driver class for Postgresql
        jdbc_driver_class => "org.postgresql.Driver"
        # our query
        statement => "SELECT * FROM logstash"
        schedule => "* * * * *"
    }
}

filter {
 mutate {
  add_field => {"[location][lat]" => "%{lat}"
                "[location][lon]" => "%{long}"}
  }

 mutate {
  convert => {
   "[location][lat]" => "float"
   "[location][lon]" => "float"
  }
 }

}

output {
    elasticsearch {
        index => "postgres_db"
        document_type => "objects_table"
        hosts => "localhost:9200"
        document_id => "%{feature_id}"

```

As you can see, the input section contains [different parameters](https://www.elastic.co/guide/en/logstash/5.3/plugins-inputs-jdbc.html) such as credentials for logging into the database, SQL statement, and a scheduler for how often the SQL statement should be run.

The `schedule => "* * * * * *"` means that logstash will run the query and perform the I/O every minute, on the minute. See the [documentation](https://www.elastic.co/guide/en/logstash/5.4/plugins-inputs-jdbc.html) for other schedule syntax.

The filter section allows you to change up the data retrieved from the SQL statement before it's output to Elasticsearch. In the above example, we are adding fields for `lat` and `lon` to match Elasticsearch syntax for mapping a [`geopoint`](https://www.elastic.co/guide/en/elasticsearch/guide/current/geopoints.html).

When running logstash with the your `.conf` file, data will be ingested into the Elasticsearch index based on your scheduler time interval.

Summary
=======
In this section, you successfully synced the data from your Postgres database into your Elasticsearch index. New data is being re-ingested into Elasticsearch every minute (or however long you specified) via Logstash, and the ES index reflects a near real-time account of the work that your digitizers are collecting.

In the final section, we'll look at how Kibana can take the data in Elasticsearch and visualize the data in really neat ways.

![alt-text]({{ site.baseurl }}/images/banner2.png)
