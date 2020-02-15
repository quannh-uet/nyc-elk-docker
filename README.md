# Analytics 2014 NYC taxi trips with ELK on Docker

## Contents

1. [Requirements](#requirements)
2. [Analytics 2014 NYC taxi trips](#analytics-2014-nyc-taxi-trips)
3. [Configuration](#configuration)
   * [JVM tuning](#jvm-tuning)
   * [Change usernames and passwords](#change-usernames-and-passwords)


## Requirements

* Docker Engine and Docker Compose or Docker Desktop
* At least 3 GB of memory
* At least 6 GB of disk space

By default, the stack exposes the following ports:
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP
* 5601: Kibana

#### Setup Docker

* [Docker Engine](https://docs.docker.com/install/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* Docker for Desktop
  * [Docker Desktop for Windows (Microsoft Windows 10)](https://docs.docker.com/docker-for-windows/install/)
  * [Docker Desktop for Mac (macOS)](https://docs.docker.com/docker-for-mac/install/)

## Analytics 2014 NYC taxi trips

#### 1. Download data
Download `nyc_taxi_data_2014.csv` data file from [2014 New York City Taxi Trips](https://quannh-public-data.s3.ap-southeast-1.amazonaws.com/kaggle/nyc_taxi_data_2014.csv) then save to `data` folder


#### 2. Build and start ELK Stack
Clone this repository, then start the stack using Docker Compose:

```console
$ docker-compose build
$ docker-compose up
```
It will be:
 - Create an index template
 - Automatic inject NYC trips data to Elasticsearch


 #### 3. Create Kibana index patterns and import saved objects
 - Login to Kibana use:
   * http://localhost:5601
   * user name: *elastic*
   * password: *changeme*


 - Create index pattern:
   * Go to [Management > Kibana > Index patterns](http://localhost:5601/app/kibana#/management/kibana/index_patterns)
   * Index pattern name: `nyc_taxi_data_*`
   * Time Filter field name: `@pickup_datetime`

- Import saved objects:
  * Go to [Management > Kibana > Saved objects](http://localhost:5601/app/kibana#/management/kibana/objects)
  * Click `Import` button, select `kibana/import-objects.ndjson` then click `Import`
  * Edit `Index Pattern Conflicts`:
    * In column `New index pattern`select `nyc_taxi_data_*` from dropdown menu.
    * Click `Confirm all changes`

#### 4. Go to Dashboard > NYC Taxi Dashboard

#### 5. Stop and remove containers, networks, images, and volumes

Use the following Docker Compose command:

```console
$ docker-compose down
```

You can also shutdown the stack and remove all persisted data by adding the `-v` flag to the above command.

```console
$ docker-compose down -v
```

#### 6. Benchmarks index time
Benchmarks index time in machine instance with SSD disk drive.

|No|CPUs|Clock (GHz)|Memory (GB)|Index time (m)|
|:-:|:-:|:-:|:-:|:-:|
|1|4|1.6|8|128|
|2|4|4.1|8|34|
|3|8|4.1|16|13|
|4|36|4.3|72|4|

## Configuration

#### JVM tuning

Update the `{ES,LS}_JAVA_OPTS` environment variable in `docker-compose.yml`.
Exam:
```yml
elasticsearch:
  environment:
    ES_JAVA_OPTS: "-Xmx1g -Xms1g"

logstash:
  environment:
    LS_JAVA_OPTS: "-Xmx1g -Xms1g"
```

#### Change usernames and passwords
- Change `ELASTIC_PASSWORD` in docker compose (`docker-compose.yml`) file
- Replace password of the `elastic` user inside the:
  * Kibana configuration file (`kibana/config/kibana.yml`)
  * Logstash configuration file (`logstash/config/logstash.yml`) and (`logstash/pipeline/logstash.conf`)
- Restart Kibana and Logstash to apply changes: `$ docker-compose restart kibana logstash`

#### Scale (updating)
