# Dockerfile for OwnTracks Recorder

This is a Dockerfile for the [OwnTracks Recorder](https://github.com/owntracks/recorder). This docker image is based on alpine for minimal footprint. 

## Configure
The `recorder.conf` file in `config` volume is used for cofiguration. The `store` volume is used for persistent location data storage.
