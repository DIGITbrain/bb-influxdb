# General

## Deployment type

Docker

## Image

Based on official image on Docker Hub: https://hub.docker.com/_/influxdb

## Licence

MIT License

## Version

2.7

## Description

InfluxDB is a time series database built from the ground up to handle high write and query loads. InfluxDB is meant to be used as a backing store for any use case involving large amounts of timestamped data, including DevOps monitoring, application metrics, IoT sensor data, and real-time analytics.

# DEPLOYMENT

General example:

```sh
docker run -d --rm \
      --name influxdb \
      -p 8086:8086 \
      -e DOCKER_INFLUXDB_INIT_MODE=setup \
      -e DOCKER_INFLUXDB_INIT_USERNAME=user \
      -e DOCKER_INFLUXDB_INIT_PASSWORD=p4sswhichisnottooshort \
      -e DOCKER_INFLUXDB_INIT_ORG=sztaki \
      -e DOCKER_INFLUXDB_INIT_BUCKET=mybucket \
      -v $PWD/data:/var/lib/influxdb2 \
      influxdb:2.7
 ```

## Parameters

|Name|Value|Description|
|-|-|-|
|Ports|`-p 8086:8086` | InfluxDB port |
|Volumes|`-v $PWD/influxdb/data:/var/lib/influxdb2`| Persist InfluxDB data |

<br/>
For additional configuration edit `/etc/influxdb/influxdb.conf`. See [1] for more details.

Available options:
- -e `DOCKER_INFLUXDB_INIT_USERNAME`
- -e `DOCKER_INFLUXDB_INIT_PASSWORD`
- -e `DOCKER_INFLUXDB_INIT_ORG`
- -e `DOCKER_INFLUXDB_INIT_BUCKET`

## Testing:

```sh
docker exec -it influx influx bucket list

docker exec -it influx influx --host http://*public-ip*:*port* bucket list

1bad8b86c272583f	mybucket	0s		86b45033231992d3
```

# Authentication

Use:
```sh
-e DOCKER_INFLUXDB_INIT_USERNAME
-e DOCKER_INFLUXDB_INIT_PASSWORD
```

## Testing

Log in with user-pass in browser: http://\<HOST\>:8086/signin

```sh
docker exec -it influx cat /etc/influxdb2/influx-configs

token = "9kU...=="

curl -G --header "Authorization: Token 9kU...==" \
      http://localhost:8086/query \
      --data-urlencode \
      "q=SHOW DATABASES"

{"results":[{"statement_id":0,"series":[{"name":"databases","columns":["name"]}]}]}

curl -v --header "Authorization: Token 9kU...==" \
      http://localhost:8086/api/v2/ping
```

# TLS

To create server certificates (`ca.crt`, `influxdb2.key`, `influxdb2.crt`), execute the following, replace the subject (value of `-subj` parameter) as needed:

```sh
mkdir certs

openssl genrsa -out certs/ca.key 4096
openssl req -x509 -new -nodes -sha256 \
      -key certs/ca.key \
      -days 3650 \
      -subj '/O=Influx Test/CN=Certificate Authority' \
      -out certs/ca.crt

openssl genrsa -out certs/influxdb2.key 2048
openssl req -new -sha256 \
      -key certs/influxdb2.key \
      -subj '/O=Influx Test/CN=Server' | openssl x509 -req -sha256 \
      -CA certs/ca.crt \
      -CAkey certs/ca.key \
      -CAserial certs/ca.txt \
      -CAcreateserial \
      -days 365 \
      -out certs/influxdb2.crt

chmod 644 certs/influxdb2.crt
chmod 644 certs/influxdb2.key
```

## Deployment

```sh
docker run -d --rm  \
      --name influx \
      -p 8888:8086
      -e INFLUXDB_HTTP_HTTPS_ENABLED=true \
      -e DOCKER_INFLUXDB_INIT_MODE=setup \
      -e DOCKER_INFLUXDB_INIT_USERNAME=user \
      -e DOCKER_INFLUXDB_INIT_PASSWORD=p4sswhichisnottooshort \
      -e DOCKER_INFLUXDB_INIT_ORG=sztaki \
      -e DOCKER_INFLUXDB_INIT_BUCKET=mybucket \
      -v $PWD/data:/var/lib/influxdb2 \
      -v $PWD/certs/influxdb2.crt:/var/lib/ssl/influxdb2.crt \
      -v $PWD/certs/influxdb2.key:/var/lib/ssl/influxdb2.key \
      influxdb:2.0.4 \
      --tls-cert "/var/lib/ssl/influxdb2.crt" \
      --tls-key "/var/lib/ssl/influxdb2.key"
 ```

### Testing

```sh
Log in with user-pass in browser: https://*localhost*:8086/signin

curl -v --insecure --header "Authorization: Token 9kU...==" \
      https://localhost:8888/api/v2/ping
```

# References

[1] https://docs.influxdata.com/influxdb/v2.0/reference/config-options/