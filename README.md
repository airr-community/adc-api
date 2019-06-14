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

## Configuration Procedure

All configuration procedures are the same for dockerized and
non-dockerized versions of these apps.

**Configuring adc-api-mongodb-repository**

The default docker-compose setup starts mongo with authentication on,
and no users exists in the default image. To setup the database, need
to decide:

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
docker run -v /disk/mongodb:/data/db -v $PWD:/dbsetup --name adc-api-mongo airrc/adc-api-mongo-repository

# Run setup script
docker exec -it adc-api-mongo mongo admin /dbsetup/dbsetup.js

# Stop mongo and get rid of name
docker stop adc-api-mongo
docker rm adc-api-mongo

# Edit docker-compose.yml and put in mapping of mongo data directory
```

**Configuring adc-api-js-mongodb**

There is one configuration file that needs to be set up to run the
API. It can be copied from its default template.

```
cd adc-api-js-mongodb
cp .env.defaults .env
emacs .env
```

**Configuring systemd**

You can set up a systemd service file on your host machine in order to
have the service automatically restart when the host machine reboots.

```
sudo cp host/systemd/adc-api.service /etc/systemd/systems/adc-api.service

sudo systemctl daemon-reload

sudo systemctl enable docker

sudo systemctl enable adc-api
```

## Deployment Procedure

### SSL

VDJServer-Repository does not handle SSL certificates directly, and is
currently configured to run HTTP internally on port 8080. It must be
deployed behind a reverse proxy in order to allow SSL connections.

### Dockerized instances

**Docker Compose Files**

## Accessing the Repository

For general users, the API is likely accessed with a GUI that hides
the technical details of communicating with the REST API. However, it
is useful to contact manually the API using the `curl` command to
verify that the service is operational.

**AIRR Data Commons API**

The top level entrypoint for the API will return a simple success status heartbeat.

```
$ curl 'http://localhost:8080/airr/v1'
{"result":"success"}
```

The info entrypoint will return version and other info about the service.

```
$ curl 'http://localhost:8080/airr/v1/info'
{"name":"adc-api-js-mongodb","description":"AIRR Data Commons API","version":"0.1.0"}
```

## Development Setup

You will need to clone down the parent project and all submodules in order to set up a local instance of vdjserver.

```
- Clone project
$ git clone https://github.com/airr-community/adc-api.git

cd adc-api

- Clone submodules
$ git submodule update --init
$ git submodule foreach git checkout master
$ git submodule foreach git pull

- Follow configuration steps listed above in the "Configuration Procedure" section of this document
```
