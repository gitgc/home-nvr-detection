home-nvr-detection
==================

This project contains the software configuration for my home NVR/ML/TPU accelerated object detection system. Largely built on [Frigate](https://frigate.video/).

Requirements
------------
* Docker
* Docker Compose
* [Coral Edge TPU USB Accelerator](https://coral.ai/)

Steps
-----

`docker-compose up -d`

This will deploy the required Frigate, MQTT and Caddy reverse proxy servers. Note, you must replace all the `"CHANGE_ME"` values to valid production ones in the `.env` file before deploying. The USB Coral TPU must be connected to system too.
