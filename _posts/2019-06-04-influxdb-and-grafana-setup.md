---
layout: post
title: "InfluxDB and Grafana setup"
date: 2019-06-04 11:46:23
categories: sysadmin database graphs
---

# Learning InfluxDB and Grafana

First thing first, you need a machine to install InfluxDB and Grafana. I've
chosen Debian 9, you may choose something else.

## InfluxDB

These following commands are condensed from [here](https://docs.influxdata.com/influxdb/v1.7/introduction/installation/). I also took a lot from [this article](https://medium.com/@ashrafur/beginning-visualization-with-grafana-and-influxdb-81701e10569d) and put my own spin on it.

```bash
~$ wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
~$ echo "deb https://repos.influxdata.com/debian stretch stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
~$ sudo apt-get update && sudo apt-get install influxdb -y
~$ sudo systemctl unmask influxdb.service
~$ sudo systemctl start influxdb
```

Now test your `influxdb` instance with something like the following:

```bash
~$ influx
Connected to http://localhost:8086 version 1.7.6
InfluxDB shell version: 1.7.6
Enter an InfluxQL query
> SHOW DATABASES
name: databases
name
----
_internal
>
```

Success! We now have InfluxDB up and running and listening on `localhost:8086`.

Now lets create a new database to play around in.

```sql
> CREATE DATABASE testdata
> SHOW DATABASES
name: databases
name
----
_internal
testdata

```

Select the database via the next command:

```sql
> USE testdata
Using database testdata
```

Lets inject some data!

```sql
> INSERT temp,Name=data1 value=1.5
> SELECT * FROM temp
name: temp
time                Name  value
----                ----  -----
1559663365410519231 data1 1.5
> EXIT
~$
```

And as you can see, running a typical `SELECT * FROM` allows you to see the date we put in _and_ the automatic time stamp.

Ok, so lets make a remote injection to our `testdata` database into the `temp` table.

Come to a command prompt like above, and make sure `curl` in installed.

```bash
~$ curl -i -XPOST 'http://localhost:8086/write?precision=s&db=testdata' --data-binary 'temp,Name=data1 value=101'
```

Lets talk about what's happening here. You are making a `POST` to the `8086` endpoint. Then you have a `precision`
declaration at `s` which means seconds. (you can change that, and I'm pretty sure it defaults to `ms` but please read the docs ;) ). You point to the `testdata` database, and inject the values like you did at the command line. Success! You now have a way to remotely add data to `InfluxDB`. (Bonus, try from remote machine to see if it works too :) )

Ok, so we now can inject data both locally and remotely. Now lets get some real data so we can start playing with our Grafana instance.

Go ahead and `wget` down the **NOAA_data.txt** file to your influxdb local machine. Then run the following command to import the data into a new database.

```bash
~$ wget https://s3.amazonaws.com/noaa.water-database/NOAA_data.txt
~$ influx -import -path=NOAA_data.txt -precision=s -database=NOAA_water_database
~$ influx
> SHOW DATABASES
name: databases
name
----
_internal
testdata
NOAA_water_database
```

Lets go ahead and explore the data, first lets see what tables we can look at:

```sql
> USE NOAA_water_database
Using database NOAA_water_database
> SHOW MEASUREMENTS
name: measurements
name
----
average_temperature
h2o_feet
h2o_pH
h2o_quality
h2o_temperature
```

And now lets look at some data from `h2o_temperature`.

```sql
> SELECT * FROM h2o_temperature LIMIT 5
name: h2o_temperature
time                degrees location
----                ------- --------
1439856000000000000 60      coyote_creek
1439856000000000000 70      santa_monica
1439856360000000000 65      coyote_creek
1439856360000000000 60      santa_monica
1439856720000000000 68      coyote_creek
```

Awesome! We now have data injected into our database and we can read from it. Now lets open up Grafana.

## Grafana

We need to install it first, if you noticed before hand we just installed InfluxDB. This was on purpose
in case you wanted to decoulpe the software. Both use HTTP to talk, so as long as they have network access
you can talk each of them. First thing first, lets install Grafana. (I'm choosing stable here because I want
this to be long lived.)

```bash
~$ echo "deb https://packages.grafana.com/oss/deb stable main"| sudo tee /etc/apt/sources.list.d/grafana.list
~$ curl https://packages.grafana.com/gpg.key | sudo apt-key add -
~$ sudo apt-get update
~$ sudo apt-get install grafana -y
```

Now set to auto-start and start now:

```bash
~$ sudo /bin/systemctl daemon-reload
~$ sudo /bin/systemctl enable grafana-server
~$ sudo /bin/systemctl start grafana-server
```

This will start the `grafana-server` process as the `grafana` user, which was created during the package installation. The default HTTP port is `3000` and default user and group is admin.

Default login and password `admin`/`admin`.

<https://localhost:3000> is where you should be able to see the grafana dashboard.

You should now default to the `Add Datasource` page. If you don't see it, click `Configuration->Data Source`
and set your values.

**NOTE**: Even though the `localhost` is grey'd out and seems to default for the URL, you have to put in the full thing: `http://localhost:8086`

Click `Save & Test` and you should have a Green bar.

Now, lets create a new Graph, click `Create->New DashBoard->Choose Visualization`.

It will give you a good amount of options, stick with Graph for now.

Click `Panel Title->Edit` so you get a Query field and edit the "A" so it looks like:

```sql
FROM default h2o_temperature WHERE +
SELECT field(degrees) mean() +
GROUP BY time($__interval) fill(null) +
FORMAT AS Time Series
ALIAS BY
```

After that you will need to change your range up in the right hand corner to Aug 16 2015 to Sept 19 2015, then click
apply. You should see your graph!

Success! You now have a way to inject data into InfluxDB, then visualize it with Grafana!
