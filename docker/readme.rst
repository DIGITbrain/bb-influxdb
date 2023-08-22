================
InfluxDB
================

About
=====

**InfluxDB** [1]_ InfluxDB is a time series database built from the ground up to handle high write and query loads. InfluxDB is meant to be used as a backing store for any use case involving large amounts of timestamped data, including DevOps monitoring, application metrics, IoT sensor data, and real-time analytics.
Source codes are available onb GitHub [2]_.

Version
-------
InfluxDB version **2.7** is deployed based on official Docker Hub image [3]_.

License
-------
**MIT License** [4]_

Pre-requisites
==============

* *docker* installed
* access to DIGITbrain private docker repo (username, password) to pull the image:

  - ``docker login dbs-container-repo.emgora.eu``
  - ``docker pull dbs-container-repo.emgora.eu/influxdb:2.7``

Usage
=====

.. code-block:: bash

$ docker run -p 8086:8086 \
      -e DOCKER_INFLUXDB_INIT_MODE=setup
      -e DOCKER_INFLUXDB_INIT_USERNAME=myusername \
      -e DOCKER_INFLUXDB_INIT_PASSWORD=mypassword \
      -e DOCKER_INFLUXDB_INIT_ORG=myorg \
      -e DOCKER_INFLUXDB_INIT_BUCKET=mybucket \
      influx:2.7


where DOCKER_INFLUXDB_INIT_USERNAME and DOCKER_INFLUXDB_INIT_PASSWORD are the username and password to access the storage,
DOCKER_INFLUXDB_INIT_ORG and DOCKER_INFLUXDB_INIT_BUCKET will be created on startup.

.. warning::
  Always update the ``DOCKER_INFLUXDB_INIT_USERNAME`` and ``DOCKER_INFLUXDB_INIT_PASSWORD`` with values of your choice
  prior to running this container.


Security
========
The image uses username-password authentication and HTTPS with a server cerificate signed by DIGITbrain CA.

Certificates in the container can be found in directory: /etc/ssl,
which can (should) be updated. See Volumes below to override.

Using MinIO client, you can disable hostname verification.

.. code-block:: bash

  $ curl -vk --cacert "ca.pem" https://<influx-db-host>:8086
  # InfluxDB CLI
  $ export INFLUX_SKIP_VERIFY="true"
  $ influx config create --config-name inf --host-url https://localhost:8086 --org myorg --token <token> --active
  $ influx ping
    OK
Configuration
-------------

Environment variables
---------------------
.. list-table::
   :header-rows: 1

   * - Name
     - Example
     - Comment
   * - *DOCKER_INFLUXDB_INIT_USERNAME*
     - ``-e DOCKER_INFLUXDB_INIT_USERNAME=myusername``
     - The username to set for the system's initial super-user (Required)
   * - *DOCKER_INFLUXDB_INIT_PASSWORD*
     - ``-e DOCKER_INFLUXDB_INIT_PASSWORD=mypasswoed``
     - The password to set for the system's inital super-user (Required)
   * - *DOCKER_INFLUXDB_INIT_ORG*
     - ``-e DOCKER_INFLUXDB_INIT_ORG=myorg``
     - The name to set for the system's initial organization (Required)
   * - *DOCKER_INFLUXDB_INIT_BUCKET*
     - ``-e DOCKER_INFLUXDB_INIT_BUCKET=mybucket``
     - The name to set for the system's initial bucket (Required)
   * - *DOCKER_INFLUXDB_INIT_RETENTION*
     - ``-e DOCKER_INFLUXDB_INIT_RETENTION=1w``
     - The duration the system's initial bucket should retain data. If not set, the initial bucket will retain data forever.

Ports
-----
.. list-table::
  :header-rows: 1

  * - Container port
    - Host port bind example
    - Comment
  * - InfluxDB API and UI (8086)
    - ``-p 10000:8086``
    - Default InfluxDB API port 8086 is opened as port HTTPS 10000 on the host.


Volumes
-------

The container might use the following volume mounts.

.. list-table::
   :header-rows: 1

   * - Name
     - Volume mount
     - Comment
   * - *Data*
     - -v $PWD/data_directory_on_host:/var/lib/influxdb2
     - InfluxDB data. Use this directory to persist data.
   * - *Configuration*
     - -v $PWD/config.yml:/etc/influxdb2/config.yml
     - InfluxDB configuration file. INFLUXD_CONFIG_PATH specificies the directory of config files.
   * - *Server key*
     - -v $PWD/certs/server-key.pem:/etc/ssl/server.key
     - Server key
   * - *Server certificate*
     - -v $PWD/certs/server-cert.pem:/etc/ssl/server.cert
     - Server key and certificate for API and Console with names: private.key and server.crt.
   * - *CA certificate*
     - -v $PWD/certs/ca.pem:/etc/ssl/public.crt
     - Certificate Authority certificate (containing server certificate and CA certificate too).

References
==========

.. [1] https://www.influxdata.com/products/influxdb-overview/

.. [2] https://github.com/docker-library/docs/blob/master/influxdb/README.md

.. [3] https://hub.docker.com/_/influxdb

.. [4] https://github.com/influxdata/influxdb/blob/master/LICENSE
