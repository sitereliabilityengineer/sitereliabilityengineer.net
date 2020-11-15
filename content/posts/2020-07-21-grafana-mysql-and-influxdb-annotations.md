---
title: Grafana MySQL and InfluxDB annotations
author: sitereal
type: post
date: 2020-07-21T19:19:06+00:00
url: /2020/07/21/grafana-mysql-and-influxdb-annotations/
pipdig_meta_geographic_location:
  - Dublin
categories:
  - Docker
  - Linux
  - Python
  - Site reliability
tags:
  - annotations
  - compose
  - docker
  - grafana
  - influxdb
  - metrics
  - mysql

---
## What are Annotations and why are useful ?



_Annotations is a way to add a Marker over the Graph (Panel). Depending on where you store the annotations, you would have the possibility to mark the Start and the End. This datasources support $start and $end in grafana:_

  1. Grafana internal datastore
  2. Elasticsearch
  3. MySQL

**Why are useful?**

It&#8217;s a simple way of highlighting and visualization an action over a Graph. For example: Somebody is running a script over any element of the infrastructure. You can add tags, and text. In the tags you can add a datacenter, server, or a change ID and in the text, you can add a description and a hyperlink. _This is very useful when you have an incident caused by a change in the infrastructure, because you will see when it started and finished the change over a metric (for example: successful logins)._ **_Obviously you need to have a good source of truth._**

**I&#8217;ve created a docker-compose environment and pushed into github:**

[https://github.com/sitereliabilityengineer/grafana\_annotations\_demo][1]

If you check the file docker-compose.yml.all you will se that we have 3 docker containers: Grafana, MySQL and InfluxDB.

To install everything execute the bash script: bootstap.sh. It will ask you for a password (to connect to the databases) and it will replace the password in all the needed files. It will take 4-7 minutes depending on your environment. It also provision the datasources and two dashboards. If you want to know how the datasources/dashboards are provisiones, take a look into $PROJECT/grafana/dashboard.yml, datasource.yml, both .json files and the Dockerfile.

The previous script will also populate both databases with some example datapoints and annotations with this two scripts: populate\_influxdb.py and populate\_mysql.py

**Grafana Access:**

URL: http://localhost:3000

The first time you access to grafana you will need to put: admin/admin (user/password) and then change it.

**There are two provisioned dashboards:**

<img class="wp-image-209" style="width: 1000px;" src="http://sitereliabilityengineer.net/wp-content/uploads/2020/07/List_of_dashboards.png" alt="" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2020/07/List_of_dashboards.png 859w, http://sitereliabilityengineer.net/wp-content/uploads/2020/07/List_of_dashboards-300x145.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2020/07/List_of_dashboards-800x386.png 800w" sizes="(max-width: 859px) 100vw, 859px" /> 

An example of both Dashboards:<figure class="wp-block-image size-large">

<img src="https://sitereliabilityengineer.net/wp-content/uploads/2020/07/mysql_dashboard.png" alt="" class="wp-image-211" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2020/07/mysql_dashboard.png 951w, http://sitereliabilityengineer.net/wp-content/uploads/2020/07/mysql_dashboard-300x143.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2020/07/mysql_dashboard-800x380.png 800w" sizes="(max-width: 951px) 100vw, 951px" /> <figcaption>MySQL Dashboard</figcaption></figure> <figure class="wp-block-image size-large"><img src="https://sitereliabilityengineer.net/wp-content/uploads/2020/07/influxdb_dashboard.png" alt="" class="wp-image-212" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2020/07/influxdb_dashboard.png 951w, http://sitereliabilityengineer.net/wp-content/uploads/2020/07/influxdb_dashboard-300x143.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2020/07/influxdb_dashboard-800x380.png 800w" sizes="(max-width: 951px) 100vw, 951px" /><figcaption>InfluxDB Dashboard</figcaption></figure> 

To stop everything you need to execute docker-compose -f docker-compose.yml.all down at the PROJECT level.

If you want to delete everything, Execute: removeAll.sh

This last script won&#8217;t delete the data inside $PROJECT/influxdb/data and you will need to do it manually.

_Please let me know if you have any question._

 [1]: https://github.com/sitereliabilityengineer/grafana_annotations_demo