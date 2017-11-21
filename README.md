InfluxDB Proxy
======

This project adds a basic high availability layer to InfluxDB.

NOTE: influx-proxy must be built with Go 1.5+, don't implement udp.

Why
---

We used [InfluxDB Relay](https://github.com/influxdata/influxdb-relay) before, but it doesn't support some demands. 
We use grafana for visualizing time series data, so we need add datasource for grafana. We need change the datasource config when influxdb is down. 
We need transfer data across idc, but Relay doesn't support gzip. 
It's inconvenient to analyse data with connecting different influxdb.
Therefore, we made InfluxDB Proxy. 

Features
--------

* Support gzip.
* Support query.
* Filter some dangerous influxql.
* Transparent for client, like cluster for client.
* Cache data to file when write failed, then rewrite.

Requirements
-----------

* Redis-server
* Python >= 2.7

Usage
------------

```sh
$ # install redis-server
$ yum install redis
$ # start redis-server on 6379 port
$ redis-server --port 6379 &
$ # Install influxdb-proxy to your $GOPATH/bin
$ go get -u github.com/alastairruhm/influx-proxy
$ # Edit config.py and execute it
$ python config.py
$ # Start influx-proxy!
$ $GOPATH/bin/influxdb-proxy -redis localhost:6379
```

Configuration
-------------

Example configuration file is at [config.py](config.py). 
We use config.py to genrate config to redis.

Description
-----------

The architecture is fairly simple, one InfluxDB Proxy process and two or more InfluxDB processes. The Proxy should point HTTP requests with measurements to the two InfluxDB servers.

The setup should look like this:

```
        ┌─────────────────┐
        │writes & queries │
        └─────────────────┘
                 │
                 ▼
         ┌───────────────┐
         │               │
         │InfluxDB Proxy │
         |  (only http)  |
         │               │         
         └───────────────┘       
                 │
                 ▼
        ┌─────────────────┐
        │   measurements  │
        └─────────────────┘
          |              |       
        ┌─┼──────────────┘       
        │ └──────────────┐       
        ▼                ▼       
  ┌──────────┐      ┌──────────┐  
  │          │      │          │  
  │ InfluxDB │      │ InfluxDB │
  │          │      │          │
  └──────────┘      └──────────┘
```

measurements match principle:

* Exact match first. For instance, we use `cpu.load` for measurement's name. The KEYMAPS has `cpu` and `cpu.load` keys.
It will use the `cpu.load` corresponding backends.

* Then Prefix match. For instance, we use `cpu.load` for measurement's name. The KEYMAPS  only has `cpu` key.
It will use the `cpu` corresponding backends.

Query Commands
--------

#### Unsupported commands

The following commands are forbid.

* `DELETE` 
* `DROP`  
* `GRANT`
* `REVOKE`

#### Supported commands

Only support match the following commands.

* `.*where.*time`
* `show.*from`
* `show.*measurements`

License
-------

MIT. 

