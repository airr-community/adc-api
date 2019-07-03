# AIRR Data Commons (ADC) API reference implementation

The AIRR Data Commons (ADC) API provides programmatic access to query
and download AIRR-seq data. The ADC API uses JSON as its communication
format, and standard HTTP methods like GET and POST.

This is a functional reference implementation for the ADC API. It
provides an API service and database on a single node. Data that
conforms to the AIRR standards can be loaded into the database, and
the API service will respond to queries and allow that data to be
downloaded.

This reference implementation lacks security features, extensive error
checking, and optimization. Therefore, this implementation should not
be directly used for a production system. However, this implementation
may be a good starting point.

## Components

The ADC API reference implementation is currently composed of 3 separate components:

 * [adc-api-mongodb-repository](https://github.com/airr-community/adc-api-mongodb-repository): MongoDB database for holding AIRR-seq data.
 * [adc-api-js-mongodb](https://github.com/airr-community/adc-api-js-mongodb): JavaScript implementation of ADC API service for MongoDB.
 * [adc-api](https://github.com/airr-community/adc-api): Compose the other components together into a complete functional repository.

## Installation Procedure

The installation and configuration of the ADC API reference
implementation requires [Docker](https://www.docker.com) and
[docker-compose](https://docs.docker.com/compose). You will need to
install them on the machine where the service will be run.

You also need a local clone of the github repositories. While images
are available on [Docker
Hub](https://cloud.docker.com/u/airrc/repository/list), configuration
files need to be defined with appropriate values. These configuration
files are external to the docker images (they may contain username,
passwords and other system sensitive information).

Clone down the parent project and all submodules then follow the
configuration procedures below.

```
# Clone project
git clone https://github.com/airr-community/adc-api.git

cd adc-api

# Init submodules
git submodule update --init --recursive
```

## Configuration Procedure

These configuration procedures only need to be performed once, after
which the service can be brought up and down whenever desired.

**Configuring MongoDB database**

The adc-api-mongodb-repository defines the docker image for the
MongoDB used by the reference implementation. The default
docker-compose setup starts MongoDB with authentication on, and no
users exists in the default image. The database files are stored on
the host computer, not in the docker container, so that data is not
lost when the container is stopped. To setup the database, need to
decide:

* Where mongo will store its files on host disk. (e.g. /disk/mongodb)

* Name of database in mongo where collections will be stored (e.g. v1airr)

* Name and password for mongo service account. This account will have
  admin privileges for managing mongo.

* Name and password for guest account. This account will only have
  read access on the database for performing queries.

Make sure not to accidently commit the dbsetup file with usernames and
passwords into the git repository.

```
# Modify dbsetup.js with appropriate settings
cd adc-api-mongodb-repository
cp dbsetup.defaults dbsetup.js
emacs dbsetup.js

# Start up temporary mongo service, note mapping of mongo data directory and dbsetup
# Replace /disk/mongodb with directory on host machine to store the database files
docker run -v /disk/mongodb:/data/db -v $PWD:/dbsetup --name adc-api-mongo airrc/adc-api-mongodb-repository

# Run setup script
docker exec -it adc-api-mongo mongo admin /dbsetup/dbsetup.js

# Stop mongo and get rid of name
docker stop adc-api-mongo
docker rm adc-api-mongo

# Edit docker-compose.yml and put in mapping of mongo data directory
```

**Configuring JavaScript API service**

The adc-api-js-mongodb defines the docker image as well as hold the
JavaScript code for the API service. There is one configuration file
that needs to be set up to run the API. It can be copied from its
default template. Set the variables for the user accounts and
passwords you defined for MongoDB.

```
cd adc-api-js-mongodb
cp .env.defaults .env
emacs .env
```

**Build the Docker Images**

The adc-api has the docker-compose configuration file which composes
the components together into a working service.

```
cd adc-api/docker-compose
docker-compose build
```

**Configuring systemd (Optional)**

You can set up a systemd service file on your host machine in order to
have the service automatically restart when the host machine
reboots. This requires root access to the host machine. If you do not
have root access, you can still run the service but you will need to
start and stop it manually.

```
# edit the file and change the directories
emacs adc-api.service

sudo cp host/systemd/adc-api.service /etc/systemd/systems/adc-api.service

sudo systemctl daemon-reload

sudo systemctl enable docker

sudo systemctl enable adc-api
```

**SSL**

VDJServer-Repository does not handle SSL certificates directly, and is
currently configured to run HTTP internally on port 8080. It should be
deployed behind a reverse proxy in order to allow SSL connections.

**Starting and Stopping the Service**

If you setup the systemd service file, then you can start and stop
with the following commands:

```
# start service
sudo systemctl start adc-api

# stop service
sudo systemctl stop adc-api
```

Otherwise, you can manually start and stop

```
# start service
cd adc-api/docker-compose
docker-compose up

# stop service
# do this from another command line
cd adc-api/docker-compose
docker-compose down
```

## Accessing the AIRR Data Commons API

For general users, the API is likely accessed with a GUI that hides
the technical details of communicating with the REST API. However, it
is useful to contact manually the API using the `curl` command to
verify that the service is operational.

**Checking the Status of the API Service**

The top level entrypoint for the API will return a simple success status heartbeat.

```
curl 'http://localhost:8080/airr/v1'
{"result":"success"}
```

The info entrypoint will return version and other info about the service.

```
curl 'http://localhost:8080/airr/v1/info'
{"name":"adc-api-js-mongodb","description":"AIRR Data Commons API","version":"0.1.0"}
```

## Load Data

The MongoDB does not come with any AIRR-seq data within it.

## Query Data
