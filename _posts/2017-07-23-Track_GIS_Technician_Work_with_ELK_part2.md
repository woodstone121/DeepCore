---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 2
author: Gina Li
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

Part 2: Use Logstash to Sync Data from PostgresDB to ElasticSearch
=======
In this section, we'll explore how to sync the geometry data and metadata that our digitizers have created in QGIS (and thus Postgres) to Elasticsearch so that we can begin using our data with the ELK stack.

First, we'll want to install [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html) and [Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html) if you haven't already. Also install the [Postgres JDBC driver](https://jdbc.postgresql.org/download.html) (download the .jar file and place it in your Java classpath) which allows you to read input from Postgres and output results to Elasticsearch. If you're having trouble, `echo $JAVA_HOME`, navigate to the output location, and then `cd` into the `lib` directory. This is where you want to drop the .jar file.

From the logstash directory (usually located in `/usr/share/logstash`, but see [the official guide](https://www.elastic.co/guide/en/logstash/5.4/dir-layout.html) for other possible directory layouts on your system), create a `postgres-to-ES.conf` file. In this file, put:

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
        document_type => "spacenet_mod"
        hosts => "localhost:9200"
        document_id => "%{feature_id}"

```

Update the `jdbc_connection_string` with your server IP address, the  `jdbc_user` and `jdbc_password` with an account with `SELECT` access, and create a Postgres `VIEW` in your table named `logstash` where your query string will live. Make sure your query returns columns `lat` and `long`, which are the x and y coordinates of the centroid of each bounding box.

The `schedule => "* * * * * *"` means that logstash will run the query and perform the I/O every minute, on the minute. See the [documentation](https://www.elastic.co/guide/en/logstash/5.4/plugins-inputs-jdbc.html) for other schedule syntax.

Save the file, and run it with from your logstash directory with `bin/logstash -f postgres-to-ES.conf`.

Executing `curl -XGET 'localhost:9200/postgres_db/digitized_polygons?pretty'` should list out your data, as it is stored in Elasticsearch, confirming that you have synced data from Postgres to Elasticsearch.

Summary
=======
In this section, you successfully synced the data from your Postgres database into your Elasticsearch index. New data is being reingested into Elasticsearch every minute, on the minute via Logstash, and reflects a near real-time account of the work that your digitizers are collecting.

By now, you have successfully set up work-flows for your digitizers to enter their data into Postgres via QGIS and synced the data from Postgres to Elasticsearch. Next, we'll look into how Kibana can take the data in Elasticsearch and visualize the data.
