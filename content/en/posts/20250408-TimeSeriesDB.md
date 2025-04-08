---
title: "Time Series Database with openHAB"
date: 2025-04-08T16:51:00Z
featured_image: ""
description: ""
tags: []
author: daniel
draft: false
---

# Time Series Database with openHAB

For historic reasons I have most of my data from [openHAB](https://www.openhab.org/) stored in a [PostgreSQL](https://www.postgresql.org/) via the [JDBC Persistence Service](https://www.openhab.org/addons/persistence/jdbc/#jdbc-persistence). The reason why is very trivial. I was on a rush, because my photovoltaic system was about to be installed pretty soon. Right from the beginning I wanted to persist all that nice and shiny data I was about to see regarding my electricity consumption and production. So for PostgreSQL there were some [ansible](https://docs.ansible.com/ansible/latest/index.html) scripts available I could use to easily spin up a database and connect to it from openHAB.

Fast forward a few years. My old PostgreSQL 11 database was really outdated and I had to take the necessary steps to upgrade to version 13. Afterwards I transferred all that historic time data over to a new dedicated host on my Proxmox cluster with an up to date debian bookworm OS and version 15 of PostgreSQL.

### VictoriaMetrics

So time to look into something more suitable for time series data. A friend proposed [VictoriaMetrics](https://victoriametrics.com/products/open-source/), which claims to be compatible with [InfluxDB](https://www.influxdata.com/). In addition there is an ansible playbook available and maintained. So very easy to setup a VictoriaMetrics database. All it takes is the following yaml definition for a single server:
```yaml
- name: Install victoriametrics time based database.
  hosts: victoriametrics
  vars:
    victoriametrics_data_dir: "/data/victoriametrics/"
    victoriametrics_retention_period_months: "1200"
  roles:
    - victoriametrics.cluster.single
```
One of the drawbacks I stumbled upon in the documentation is, that by default the single server does not have any authentication or authorization in place. So this would need additional efforts by e.g. setting up a reverse proxy in front of the server.

My main issue however was that the integration as data store for openHAB worked easy and flawlessly. Just configuring it as a InfluxDB v1 service in influxdb.cfg. <br>
Retrieving of data for display of the collected data in openHAB to the contrary unfortunately never worked. I just saw a whole lot of error messages regarding http (I believe, unfortunately quite some time passed between me trying and the writeup here) endpoints. <br>
Another critical issue for me was the constant stream of disk I/O and network traffic the service produced within my Proxmox cluster. That even seemed to be the case after switching openHAB over to InfluxDB. To be fair, the "do not store the database on anything than bare metal" was openly mentioned in the documentation.

Screenshot of VictoriaMetrics resource utilization while completely idle:
![VictoriaMetrics resource utilization completely idle](/images/2025/VictoriaMetricsIdling.png)

### InfluxDB v2

So to the rescue came an InfluxDB v2 server. Maybe at a later point in time I will also publish the simple ansible scripts it needed to spin up the server and provision initial user and password as well as organisations and buckets. <br>
The gain is a really easy integration with openHAB as data sink and data source. Only the migration from PostgreSQL data to InfluxDB is still open, as the existing scripts I found unfortunately seem to be outdated.

Screenshot of InfluxDB v2 resource utilization with real time data congestion of my full house (temperatures, energy, ...):
![InfluxDB v2 resource utilization with real time data congestion of my full house (temperatures, energy, ...)](/images/2025/InfluxDB_Ingesting_openHAB_data.png)
