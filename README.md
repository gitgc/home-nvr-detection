# home-nvr-detection

This project contains the software configuration for my home NVR/ML/TPU accelerated object detection system. Largely built on [Frigate](https://frigate.video/).

## Architecture

```mermaid
graph TB
    subgraph Cameras["IP Cameras"]
        C1[entrance]
        C2[workshop-yard]
        C3[garage-yard]
        C4[drive]
        C5[garden]
    end

    subgraph Stack["Docker Compose Stack"]
        Scrypted["Scrypted<br>RTSP Proxy / Transcoder"]
        Frigate["Frigate NVR<br>go2rtc · Object Detection<br>Face Recognition · LPR"]
        MQTT["Eclipse Mosquitto<br>MQTT Broker"]
        Caddy["Caddy<br>Reverse Proxy + TLS"]
    end

    subgraph Client["Client"]
        User(["Browser / Mobile App"])
    end

    TPU["Coral Edge TPU × 2<br>USB Accelerator"]
    GPU["Intel GPU<br>QSV H.264 Decode"]
    Disk[("NVR Storage<br>/nvr-media/frigate")]

    C1 & C2 & C3 & C4 & C5 -->|RTSP| Scrypted
    Scrypted -->|"RTSP restream<br>host.docker.internal"| Frigate
    Frigate <-->|events| MQTT
    TPU & GPU --> Frigate
    Frigate --> Disk

    User -->|"HTTPS :443"| Caddy
    Caddy -->|"nvr.*"| Frigate
    Caddy -->|"scrypted.*"| Scrypted

```

## Requirements

* Docker
* Docker Compose
* [Coral Edge TPU USB Accelerator](https://coral.ai/)

## Steps

### 1. Configure the `.env` file with your environment variables

`cp .env.example .env`

Edit the `.env` file to set your environment variables.

### 2. Generate the Mosquitto password file

Generate the Mosquitto password file with the following command:

`docker-compose run --rm mqtt mosquitto_passwd -c /mosquitto/mosquitto.passwd ${FRIGATE_MQTT_USER}`

This will prompt you to enter a password for the MQTT user specified in the `.env` file. This will create the `mosquitto.passwd` file with the appropriate credentials for MQTT authentication, which is used by both the MQTT and Frigate containers.

### 3. Deploy the stack

`docker-compose up -d`

This will deploy the required Frigate, MQTT and Caddy reverse proxy servers. Note, you must replace all the `"CHANGE_ME"` values to valid production ones in the `.env` file before deploying, and setup the MQTT password file. The USB Coral TPU must be connected to system too.
