# Dockerfile for OwnTracks Recorder
[![Build Status](https://travis-ci.com/deisi/docker-ot-recorder.svg?branch=master)](https://travis-ci.com/deisi/docker-ot-recorder)

Dockerfile for the [Recorder](https://github.com/owntracks/recorder) of the
Owntracks project.

## Quickstart
```bash
$ docker volume create recorder_store
$ docker run -d -p 8083:8083 -v recorder_store:/store -e OTR_HOST=mqtt_broker owntracks/recorder
```

Recorder is now accessible at (http://localhost:8083). 

`-p 8083:8083` makes the container reachable at port 8083. `-d` detaches the
container into the background. The volume `recorder_store` is mounted at
`/store` into the container. This is needed to have persistent data storage.
`-e` allows to pass additional configuration to the container as
environment variables. Multiple `-e` parameters can be used for multiple
environment variables.

## Configuration
The Recorder can be configured using two methods, environment variables and
via the a `recorder.conf` file in the `/config` volume of the container.

### Environment variables
Can be passed to the container with the `-e` parameter. Example:
```bash
$ docker run -d -p 8083:8083 -e OTR_HOST=mqtt_broker -e OTR_PORT=1883 -e OTR_USER=user -3 OTR_PASS=pass owntracks/recorder 
```

The complete list of parameters can be found in the [recorder
documentations](https://github.com/owntracks/recorder/blob/master/README.md#configuration-file)

### Configuration file
One can also use a configuration file. The container reads a `recorder.conf`
file from the `/config` folder. To use this, create a folder e.g. `./config` and
mount it into you docker container at `/config`.

```bash
$ mkdir config
$ docker run -d -p 8083:8083 -v recorder_store:/store -v ./config:/config owntracks/recorder
```

Up on starting the recorder, a default `recorder.conf` file will be created if
none exists. This file can then be added to ones liking. Possible options are
documented
[here](https://github.com/owntracks/recorder/blob/master/README.md#configuration-file)

**Note**: Environment variables, overwrite the `recorder.conf` file options.

## Storing data
The `/store` volume of the container is used for persistent location data
storage. The volume needs to be created explicitly to have persistent data
storage.

```
$ docker volume create storage
$ docker run -d -p 8083:8083 -v store:/store owntracks/recorder
```
It is also possible to use a local folder instead of an static docker volume.
```bash
& mkdir store
$ docker run -d -p 8083:8083 -v ./store:/store owntracks/recorder
```

If nothing is mounted at `/store`, docker will create a unique volume
automatically. However up on recreation of the docker container, this process
will be repeated and another unique volume will be created. As a result, the
container will have forgotten about previous tracks.

## Docker compose files
Save a file with the name `docker-compose.yml` and following content. Run it
with `docker-compose up` from the same folder.

``` yaml
version: '3'

services:

  otrecorder:
    image: owntracks/recorder
    ports:
      - 8083:8083
    volumes:
      - config:/config
      - store:/store
    restart: unless-stopped

volumes:
  store:
  config:

```

This `docker-compose.yml` file creates `store` and `config` volumes. It is
possible to edit the `recorder.conf` file in the `config` volume to get your
setup up and running. It is also possible to pass environment variables to the
docker container via the `environment:` keyword. For details see
[here](https://docs.docker.com/compose/environment-variables/) and for available
variables see
[here](https://github.com/owntracks/recorder/blob/master/README.md#configuration-file).

An example might look like:

``` yaml
version: '3'

services:

  otrecorder:
    image: owntracks/recorder
    ports:
      - 8083:8083
    volumes:
      - store:/store
    restart: unless-stopped
    environment:
      - OTR_HOST=mqtt_broker
      - OTR_USER=user
      - OTR_PASS=pass

volumes:
  store:

```

If an mqtt broker is needed mosquitto can be used. There are ready to use
containers available on docker hub. To use `eclipse-mosquitto` add the following
to you `docker-compose.yml` file.

``` yaml
version: '3'

services:

  otrecorder:
    image: owntracks/recorder
    ports:
      - 8083:8083
    volumes:
      - config:/config
      - store:/store
    restart: unless-stopped

  mosquitto:
    image: eclipse-mosquitto
    ports:
      - 1883:1883
      - 8883:8883
    volumes:
      - mosquitto-data:/mosquitto/data/
      - mosquitto-logs:/mosquitto/mosquitto.log
    configs:
      - mosquitto-conf:/mosquitto/config/mosquitto.conf
    restart: unless-stopped

volumes:
  store:
  config:
  mosquitto-data
  mosquitto-logs
configs:
  mosquitto-conf
```
See [here](https://hub.docker.com/_/eclipse-mosquitto) for info on the eclipse-mosquitto image and how to configure.

# Notes
- certificates volume for OT-Recprder and some documentation
- Check if lua5.2-libs is needed
- Check if eclipse-mosquitto is working
- Maybe put the most common mosquitto options
- Maybe add letsencrypt
- Add some defaults so mosquitto and owntracks work right away?
- Do we really want to use `configs` for mosquitto setup?

