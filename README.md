# Dockerfile for OwnTracks Recorder
[![Build Status](https://travis-ci.com/deisi/docker-ot-recorder.svg?branch=master)](https://travis-ci.com/deisi/docker-ot-recorder)

This is a Dockerfile for the [OwnTracks Recorder](https://github.com/owntracks/recorder)(OT-Recorder).

## Using the recorder image
```
$ docker run -d -p 8083:8083 owntracks/recorder
```
After this you can access the Recorder at (http://localhost:8083).

## Configuration
The OT-Recorder can be configured using two methods, environment variables and
via the a `recorder.conf` file in the `config` volume of the container.

### Environment variables
Can be passed to the container with the `-e` parameter. Example:
```
$ docker run -d -p 8083:8083 -e OTR_HOST=mqtt_broker -e OTR_PORT=1883 -e OTR_USER=user -3 OTR_PASS=pass owntracks/recorder 
```
The complete list of parameters can be found in the [recorder documentations](https://github.com/owntracks/recorder/blob/master/README.md#configuration-file)

### Configuration file
One can also use a configuration file. The image reads a `recorder.conf` file
from the `config` folder. Possible options are documented
[here](https://github.com/owntracks/recorder/blob/master/README.md#configuration-file)

**Note**: Environment variables, overwrite the `recorder.conf` file.

## Storing data
The `store` volume is used for location data storage. Create the volume
explicitly if you want to have persistent data storage.

```
$ docker volume create storage
$ docker run -d -p 8083:8083 -v store:/store owntracks/recorder
```

## Docker compose files
Save a file with the following content and run with `docker-compose up`

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

This `docker-compose.yml` file creates the `store` and `config` volumes for you.
You can edit the `recorder.conf` file in the `config` volume to get your
setup up and running. If you prefer environment variables to configure
OT-Recorder you can do so. For details on how to do that see
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

If you need a mqtt broker add the following to you `docker-compose.yml` file.

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
See [here](https://hub.docker.com/_/eclipse-mosquitto) for info on the eclipse-mosquitto image.

# Notes
- certificates volume for OT-Recprder and some documentation
- Check if lua5.2-libs is needed
- Check if eclipse-mosquitto is working
- Maybe put the most common mosquitto options
- Maybe add letsencrypt
- Add some defaults so mosquitto and owntracks work right away?
- Do we really want to use `configs` for mosquitto setup?

