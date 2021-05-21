# IMAGE SOURCE

Official image on __Docker Hub__:  https://hub.docker.com/_/influxdb

# Licence

MIT License

# Version

2.0.4

# DEPLOYMENT

Example:

```text
docker run -p 8086:8086 \
      --name influx \
      -d --rm \
      -v $PWD/data:/var/lib/influxdb2 \
      -e DOCKER_INFLUXDB_INIT_MODE=setup \
      -e DOCKER_INFLUXDB_INIT_USERNAME=user \
      -e DOCKER_INFLUXDB_INIT_PASSWORD=p4sswhichisnottooshort \
      -e DOCKER_INFLUXDB_INIT_ORG=sztaki \
      -e DOCKER_INFLUXDB_INIT_BUCKET=mybucket \
      influxdb:2.0.4
 ```

VOLUMES:

        -v $PWD/data:/var/lib/influxdb2

OPTIONS:

      -e DOCKER_INFLUXDB_INIT_USERNAME=user 
      -e DOCKER_INFLUXDB_INIT_PASSWORD=p4sswhichisnottooshort 
      -e DOCKER_INFLUXDB_INIT_ORG=sztaki 
      -e DOCKER_INFLUXDB_INIT_BUCKET=mybucket 

CONFIGURATION:

See: https://docs.influxdata.com/influxdb/v2.0/reference/config-options


See also: https://github.com/influxdata/influxdata-docker/blob/c3c73281e6c0dfa14383c1cf5cd4fb6a7521872a/influxdb/2.0/

## TEST:

> $ docker exec -it influx influx bucket list
>
> $ docker exec -it influx influx --host http://*public-ip*:*port* bucket list
>
>> 1bad8b86c272583f	mybucket	0s		86b45033231992d3

# AUTHENTICATION

Use:

        -e DOCKER_INFLUXDB_INIT_USERNAME
        -e DOCKER_INFLUXDB_INIT_PASSWORD

## TEST

> Log in with user-pass in browser: http://*localhost*:8086/signin
>
> docker exec -it influx cat /etc/influxdb2/influx-configs
>
>> token = "9kU...=="
>
> $ curl -G --header "Authorization: Token 9kU...==" http://localhost:8086/query --data-urlencode "q=SHOW DATABASES"
>> {"results":[{"statement_id":0,"series":[{"name":"databases","columns":["name"]}]}]}
>
>  $ curl -v --header "Authorization: Token 9kU...==" http://localhost:8086/api/v2/ping


# TLS over HTTPS

Use: 

    --tls-cert
    --tls-key


> mkdir certs

> openssl genrsa -out __certs/ca.key__ 4096

> openssl req -x509 -new -nodes -sha256 -key certs/ca.key -days 3650 -subj '/O=Influx Test/CN=Certificate Authority' -out __certs/ca.crt__

> openssl genrsa -out __certs/influxdb2.key__ 2048

> openssl req -new -sha256 -key certs/influxdb2.key -subj '/O=Influx Test/CN=Server' | openssl x509 -req -sha256 -CA certs/ca.crt -CAkey certs/ca.key -CAserial certs/ca.txt -CAcreateserial -days 365 -out __certs/influxdb2.crt__

> chmod 644 certs/influxdb2.crt

> chmod 644 certs/influxdb2.key


__NOTE__: run after the above command, stop it, then run as below (otherwise init goes infinite ping loop)! 
See: https://github.com/influxdata/influxdata-docker/issues/467

```text
docker run -p 8888:8086 \
      --name influx \
      -d --rm \
      -v $PWD/data:/var/lib/influxdb2 \
      -e INFLUXDB_HTTP_HTTPS_ENABLED=true \
      -v $PWD/certs/influxdb2.crt:/var/lib/ssl/influxdb2.crt \
      -v $PWD/certs/influxdb2.key:/var/lib/ssl/influxdb2.key \
      -e DOCKER_INFLUXDB_INIT_MODE=setup \
      -e DOCKER_INFLUXDB_INIT_USERNAME=user \
      -e DOCKER_INFLUXDB_INIT_PASSWORD=p4sswhichisnottooshort \
      -e DOCKER_INFLUXDB_INIT_ORG=sztaki \
      -e DOCKER_INFLUXDB_INIT_BUCKET=mybucket \
      influxdb:2.0.4 \
      --tls-cert "/var/lib/ssl/influxdb2.crt" --tls-key "/var/lib/ssl/influxdb2.key"
 ```

## TEST

> Log in with user-pass in browser: __https__://*localhost*:8086/signin

> curl -v --insecure --header "Authorization: Token GL3bMyHxgMRqsoqEmtpNxucdpydM7irJK8g-DzaTNkNWDEJwyWe7fSyxDAh_VmO9WLsAKYkUK8Iefm8LgV2-4g==" https://localhost:8888/api/v2/ping


# TLS mutual authentication

N/A

