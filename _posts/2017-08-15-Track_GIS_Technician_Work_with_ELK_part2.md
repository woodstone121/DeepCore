---
layout: post
title: Track GIS Technician Work with the ELK Stack - Part 2
author: Gina Li
author_title: Data Scientist Intern
desc: How we tracked and visualized employee digitizing metrics using elasticsearch, logstash, and kibana
keywords: elasticsearch, logstash, kibana, postgres, postgis, visualization, dashboard, digitizing, metrics, track
---

In [Part 1]({{ site.baseurl}}/2017/08/11/Track_GIS_Technician_Work_with_ELK_part1.html), I introduced the concept of monitoring a GIS digitizing project in near real time with a website dashboard using the ELK ([Elasticsearch](https://www.elastic.co/products/elasticsearch), [Logstash](https://www.elastic.co/products/logstash), and [Kibana](https://www.elastic.co/products/kibana)) stack. A dashboard can tell you up-to-the-minute details about how many objects have been digitized, how many of each type have been digitized, where they've been digitized, and productivity metrics of the digitizers themselves like hourly rate and cumulative counts per employee.

![dashboard_example]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/dashboard_example.png){: width="100%"}
*The above dashboard shows project metrics with visualizations created using Kibana. Names redacted/removed to preserve anonymity.*

We began with the first step,  storing the data created by our digitizers in a remote database for easy access. By connecting each user's QGIS workspace to the server database, you have the ability as a manager to watch the database grow as data is added.

![Digitizer_Pipeline]({{ site.baseurl }}/assets/images/2017-07-20-Track_GIS_Technician_Work_with_ELK/Digitizer_Pipeline.png){: width="100%"}

Querying the database for results is fine and dandy if you're a DB pro, but we want to have visualizations that will show our results instead. This is Part 2, where we discuss syncing the database to Elasticsearch so we can actually utilize the Elastic (aka ELK) stack to create the visualizations.

Part 2: Use Logstash to Sync Data from Database to Elasticsearch
=======

Why do we need to get our data into Elasticsearch if we already have it stored in our Postgres database? The short answer is because Kibana will only draw data from Elasticsearch (hence why it's called the ELK stack). These components must work TOGETHER - Kibana is the user interface that allows you to build pretty much any visualization you'd like with the data from Elasticsearch with zero coding. Luckily, there's also Logstash, which provides the input stream to Elasticsearch. Logstash can connect to our database and munge the data to fit the syntax and data structure that Elasticsearch can store and query for Kibana visualizations. Thus, each component of the ELK stack is equally important for full implementation.

You'll need to be familiar with Elasticsearch syntax. Additionally, queries are made through an HTTP web interface, or on the CLI with `curl`. See the [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/_introducing_the_query_language.html) for more information on querying Elasticsearch.

Here's a snippet of how documents are stored in Elasticsearch. This is the output after performing a search query via a `GET` request:

```
{
        "_index": "postgres_db",
        "_type": "digitizing_table",
        "_id": "1",
        "_score": 1,
        "_source": {
          "feature_id": 1,
          "ingest_time": "2017-07-28T13:40:26.915Z",
          "@timestamp": "2017-07-28T13:41:33.910Z",
          "@version": "1",
          "location": {
            "lon": -90.5262838841928,
            "lat": 14.602028346562024
          },
          "object": "evergreen_tree",
          "long": -90.5262838841928,
          "lat": 14.602028346562024,
          "edited_by": "tommy"
        }
      },
      {
        "_index": "postgres_db",
        "_type": "digitizing_table",
        "_id": "2",
        "_score": 1,
        "_source": {
          "feature_id": 2,
          "ingest_time": "2017-07-28T13:40:51.943Z",
          "@timestamp": "2017-07-28T13:41:33.910Z",
          "@version": "1",
          "location": {
            "lon": -90.52630504075606,
            "lat": 14.602240390435368
          },
          "object": "maple_tree",
          "long": -90.52630504075606,
          "lat": 14.602240390435368,
          "edited_by": "johnny"
        }
      },

      ...

```
*Notice the JSON object format, where each object specifies which `index` and `type` the document is a part of. In Elastic stack terms, think index, mapping, type, and document synonymously to database, schema, table, and row (respectively) from an RDMS standpoint.*

Elasticsearch stores schema-free JSON documents. All documents are indexed, which makes searching incredibly fast, even with huge amounts of data. A common application for Elasticsearch is scraping and storing data from internet search results such as Wikipedia articles or Twitter tweets for keywords (i.e. how often do the words "machine learning" get mentioned?). Needless to say, Elasticseach is built and written to handle quick searches on large volumes of data.

Let's set up Logstash to build our index. Since our input is from a database, make sure you have the correct JDBC driver installed for Logstash to work. A JDBC driver enables a Java-based application (Logstash) to interact with a database. For a [Postgres](https://www.postgresql.org/download/) database, use the associated [Postgres JDBC driver](https://jdbc.postgresql.org/download.html). Make sure you place the JDBC driver in your Java classpath (to do this, `echo $JAVA_HOME`, navigate to the output location, then drop the .jar file in the `lib` subdirectory).

A minimal logstash config file for our purposes will look something like this:

```
input {
    jdbc {
        # Postgres JDBC connection string to your database
        jdbc_connection_string => "jdbc:postgresql://<YOUR SERVER>:5432/<YOUR DB>"
        # User credentials to connect to Postgres
        jdbc_user => "ginal"
        jdbc_password => "password"
        # Path to downloaded JDBC driver in Java classpath
        jdbc_driver_library => "/usr/lib/jvm/java-8-openjdk-amd64/jre/lib/postgr
esql-42.1.1.jre6.jar"
        # The name of the JDBC driver class for Postgres
        jdbc_driver_class => "org.postgresql.Driver
        # SQL query to execute
        statement => "SELECT * FROM logstash"
        schedule => "* * * * *"
    }
}


output {
    elasticsearch {
        # Give the index a name, convention is the same name as your db
        index => "postgres_db"
        # Give the document type a name, convention is the same name as your table
        document_type => "digitizing_table"
        hosts => "localhost:9200"
```

The input and output sections contain [different parameters](https://www.elastic.co/guide/en/logstash/5.3/plugins-inputs-jdbc.html) such as credentials for logging into the database and SQL statement. You can even add a scheduler for how often the SQL statement should be run.

Basically what happens behind the scenes is that Logstash will connect to the database, perform the SQL query and create a document per row. Each document contains attributes that correspond to the columns. Logstash also makes its best guess at the data type of each attribute.

Additionally, you can add a filter between the input and output which allows you to change up the data retrieved from the SQL statement before it's output to Elasticsearch. Since we care to visualize locations containing latitude and longitude, here's an example of a filter that could be added to create a location mapping to the Elasticsearch index:

```
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
```

To enable to utilize the `geopoint` datatype, you'll need to add a `location` mapping to Elasticsearch too, which like an RDMS schema, describes the fields that documents of a particular type can have.

```
PUT digitizing_table
{
  "mappings": {
    "my_type": {
      "properties": {
        "location": {
          "type": "geo_point"
        }
      }
    }
  }
}
```
*Note: you'll want to run the `PUT` command before Logstash or else Logstash will complain that the index already exists*

See the [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html) for more information about the `location` mapping and the `geopoint` datatype.

Once you have all the configurations set up, running Logstash will ingest data into Elasticsearch every minute (or however often you've scheduled it to). Logstash runs as a service, so it will continue to work even after you close your connection.

Summary
=======
In this section, we explained how Logstash is the intermediary between your database and your Elasticsearch index. Logstash is important for syncing your data in near real time with the Elastic stack.

In the final (and most fun) section, we'll make some cool visualizations in Kibana with the data that is not in Elasticsearch.

![alt-text]({{ site.baseurl }}/images/banner2.png)
