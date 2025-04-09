
# Deploy to Balena

[![balena deploy button](https://www.balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy)

The default device variables setup from `balena.yml` should be correct but these work here

![image](https://github.com/DynamicDevices/openthread-border-router-block/assets/1537834/e3ae3e78-527d-425d-82e8-d3412602fa55)

NOTE: To enable USB ports on CM4 I/O Board you need to set

```
DT overlays - add "dwc2"
DT parameters - "dr_mode=host"
```

# Overview

This repository provides a container for the OpenThread Border Router.

We build it using a modified Dockerfile taken from the OTBR repository [here](https://github.com/openthread/ot-br-posix/blob/main/etc/docker/Dockerfile)

- This is to ensure we have control over the settings we use for the build.

We also include the MQTT-SNGateway based on the block [here](https://github.com/DynamicDevices/mqttsn-gateway-block)

- TODO: This is because I haven't yet worked out how to route IPv6 packets successfully between containers to make them available to the mesh on interface `wpan0`

See the MQTT-SNGateway [README](https://github.com/DynamicDevices/mqttsn-gateway-block#readme) for details on default environment variable settings.

In addition you need to set `RUN_MQTT_SNGATWAY=1` to start up the gateway process.

Then the OpenThread Border Router web interface will run up on port 80 and you can `Form` or `Join` a network.

Use the settings in this image to conform to the MQTT-SN publication CLi example you can use [here](https://github.com/DynamicDevices/openthread/tree/ajl/adding-examples/examples/apps/mqtt-snpublish)

- PAN 0x4444
- Extended PAN 3333333344444444
- Network Key (used to be called Master Key) 33334444333344443333444433334444
- Channel 15

If you want to automatically form a network you can set Balena Device Variables

| Variable | Description | Default |
| -------- | ----------- | ------- |
| AUTO_FORM_NETWORK | Automatically form on startup | 1 |
| THREAD_CHANNEL | | 15 |
| THREAD_PAN_ID | | 0x4444 |
| THREAD_EXT_PAN_ID | | 3333333344444444 |
| THREAD_NETWORK_NAME | | INST |
| THREAD_NETWORK_KEY | | 33334444333344443333444433334444 |
| THREAD_IPV6_PREFIX: | | fd11:22::/64 pasor |

![image](https://github.com/DynamicDevices/openthread-border-router-block/assets/1537834/4c6f6e93-cbde-4bdd-b5a5-1df614e700c6)

# Building CLI test firmware for Nordic nrf52840

Until we have multicast advertising and `SEARCHGW` working properly you will need to go to the container and look at `ifconfig` to work out the IP address

Here we can take the IP address of the wpan0 interface

![image](https://github.com/DynamicDevices/openthread-border-router-block/assets/1537834/62cedf1d-e734-4ced-89f8-8b75823f9086)

Then compile the MQTT-SN enabled CLI publishing example as follows

```
$ git clone https://github.com/DynamicDevices/ot-nrf528xx.git
```

![image](https://github.com/DynamicDevices/openthread-border-router-block/assets/1537834/ce1777e1-8395-449b-b9f4-b07f255027f0)

```
$ ./script/build nrf52840 USB_trans -DOT_BOOTLOADER=USB -DOT_MQTT=ON -DOT_JOINER=ON -DOT_RCP_RESTORATION_MAX_COUNT=0 -DOT_LOG_LEVEL=WARN -DOT_UPTIME=ENABLED -DUDP_FORWARD=ON
$ arm-none-eabi-objcopy -O ihex build/bin/ot-cli-ftd-mqttsn-publish build/bin/ot-cli-ftd-mqttsn-publish.hex
$ arm-none-eabi-objcopy -O ihex build/bin/ot-cli-ftd-mqttsn-searchgw build/bin/ot-cli-ftd-mqttsn-searchgw.hex
```

Then use the Nordic programmer to program your dongle with the hex file

Running the `ot-cli-ftd-mqttsn-searchgw` CLI code against the OTBR running the MQTT-SN Gateway results in serial logs like this

![image](https://github.com/DynamicDevices/openthread-border-router-block/assets/1537834/e8955a03-f387-4021-9bd7-b4211f820551)

# MING (MQTT, InfluxDB, NodeRED and Grafana)

This is an educative project called *MING* based on MQTT, InfluxDB, [balena Node-RED block](https://github.com/balenablocks/balena-node-red), Grafana and WiFi Connect balena blocks. MING is an Open-Source LAMP-like stack for the Internet of Things.

Each of these applications is built and runs in its own container on an embedded Linux target supporting balena.io or Docker:

* Mosquitto MQTT broker listening on port 1883 for MQtt message publications.
* InfluxDB listening on port 8086 providing a time series database for sensor data storage.
* NodeRed listening on port 80 to provide an easy to use graphical environment for parsing, analysing, storing, and forwarding sensor data messages.
* Grafana listening on port 8080 providing a data visualisation environment for sensor data.
* WiFi Connect listening on the port 80 in case there is not WiFi connectivity available.


## Requirements

### Hardware

* Raspberry Pi 0/2/3/4 or [balenaFin](https://www.balena.io/fin/)
* SD card in case of the RPi 3/4
* Power supply and (optionally) ethernet cable

### Software

* A balenaCloud account ([sign up here](https://dashboard.balena-cloud.com/))
* [balenaEtcher](https://balena.io/etcher)


## Deploy

You have two options here:

### One-click deploy via [Balena Deploy](https://www.balena.io/docs/learn/deploy/deploy-with-balena-button/)

You can deploy this project to a new balenaCloud application in one click using the button below:

[![](https://balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/mpous/ming)

Or, you can create an application in your balenaCloud dashboard and balena push this code to it the traditional way.

### In-control deploy via [balena CLI](https://www.balena.io/docs/reference/balena-cli/)

If you are a balena CLI expert, feel free to use balena CLI. This option lets you configure in detail some options, like adding new services to your deploy or configure de DNS Server to use.

- Sign up on [balena.io](https://dashboard.balena.io/signup)
- Create a new application on balenaCloud.
- Add a new device and download the image of the BalenaOS it creates.
- Burn and SD card (if using a Pi), connect it to the device and boot it up.

While the device boots (it will eventually show up in the Balena dashboard) we will prepare de services:

```
cd ~/workspace
git clone https://github.com/mpous/ming
cd ming
```

- Using [Balena CLI](https://www.balena.io/docs/reference/cli/), push the code with `balena push <application-name>`
- See the magic happening, your device is getting updated 🌟Over-The-Air🌟!


## Variables

### Device Variables

Variable Name | Default | Description
------------ | ------------- | -------------
PORT | `80` | the port that exposes the Node-RED UI
USERNAME | `balena` | the Node-RED admin username
PASSWORD | `balena` | the Node-RED admin password
ENCRIPTION_KEY | `balena` | the encription key used to store your credentials files

You **must** set the `USERNAME` and `PASSWORD` environment variables to be able to save or run programs in Node-RED.  


## Node-RED

For running Node-RED, use the local IP address on port 80, if you are on the same network than your device. You also can use the `Publick Device URL` by balena to access to the Node-RED UI.

### Add new Node-RED nodes

Add new Node-RED nodes on the Dockerfile templates on the `node-red` folder in the project. Find more an example [here](https://github.com/mpous/ming/blob/3c2e5eac92d7be3b643ca4fe6d29d0aefd533832/node-red/Dockerfile.raspberrypi4-64#L11).

### Add new services

For adding new services, use the `docker-compose` [here](https://github.com/mpous/ming/blob/master/docker-compose.yml). Go to [balenaHub](https://hub.balena.io) and use the blocks available there to accelerate your development.

### Deploy your flow into your entire fleet

If you would like to deploy your flow into your entire fleet, you can introduce a file into the folder `node-red/app/flows` [here](https://github.com/mpous/ming/tree/master/node-red/app/flows) on your cloned github repository. Then using the `balena CLI` call `balena push <your Fleet name>` and you will find the flow deployed on all your fleet.

Go to the Node-RED UI and `Import` the flow and start using it.



## Attribution

- This is based on the [balena Node-RED block](https://github.com/balenablocks/balena-node-red) made by Carlo Curinga and others. And also inspired by the [Razikus Node-RED](https://github.com/Razikus/balena-nodered) project. It is also inspired by [Alex Lennon MING project](https://github.com/DynamicDevices/ming).
- This is in joint effort between Carlo Curinga and Marc Pous to present in the [Node-RED Con 2021](https://nodered.jp/noderedcon2021/index-en.html).

## Disclaimer

This project is for educational purposes only. Do not deploy it into your premises without understanding what you are doing. 

# Overview

This is a Balena [Block](https://www.balena.io/blog/balenablocks-public-roadmap/) project to enable flexible management of on-device networking using [Traefik](https://traefik.io/)

# Block Configuration

This is an initial example which will configure a node-exporter service on an internal port :9100 which is then routed by Traefik to be on a path of the public URL on :80

Add the following to a `docker-compose.yml` for your fleet.

**NOTE:** I couldn't get this working for the raspberrypi4-64 which is what I am testing here. It appears to be related to an issue with Balena `--platform` parsing and I had to implement the fix shown [here](https://github.com/balena-io/balena-cli/issues/1408).

```
version: '2'

services:
  node-exporter:
    # Use a specific hash of the node-exporter (which you can find on Docker Hub) as we don't currently pick up architectures correctly
    image: dynamicdevices/balenablock-node-exporter@sha256:9c03ad3c3c7b6201c0f5a644d5d7023e8ecb42767c55945082807e16e4eb60de
    # Restart the container if there's a problem
    restart: always
    # No needfor enhanced privileges
    privileged: false
    ports:
    # The standard node-exporter port is 9100
      - '9100:9100'
    labels:
      # Enable traefik support for this container
      - "traefik.enable=true"
      # Make it available on the "web" entrypoint which is defined below as :80
      - "traefik.http.routers.node-exporter.entrypoints=web"
      # Add a specific path prefix that will be expected on the URL suffix
      - "traefik.http.routers.node-exporter.rule=PathPrefix(`/metrics`)"
  traefik:
    image: dynamicdevices/balenablock-traefik@sha256:072770347a92b1828efca812756051c1652333247fec1e56f7de3077c0b59e7e
    container_name: traefik
    command:
      - "--log.level=DEBUG"
      - "--api.dashboard=true"
      # Uncomment this to enable the Traefik WebUI on port :8080
      #- "--api.insecure"
      - "--providers.docker=true"
      - "--providers.docker.endpoint=unix:///var/run/balena-engine.sock"
      #- "--providers.docker.exposedbydefault=true"
      - "--entrypoints.web.address=:80"
    restart: always
    ports:
      # The HTTP port
      - "80:80"
      # The Dashboard port
      - "8080:8080"
    labels:
      # So that Traefik can listen to the Docker events
      - "io.balena.features.balena-socket=1"
```

With this Fleet configuration deployed you should be able browse to your public endpoint (as seen on the device dashboard)

If  you browse to the base URL you will see an error but if you browse to https://my-balena-device-id.balena-devices.com/metrics your requested will be proxied by Traefik to the node-exporter webserver running on the internal port :9100 on the device

You'll then see a list of key value pairs of device metrics)

![image](https://user-images.githubusercontent.com/1537834/157541771-a4f65193-6799-404a-9a62-767322fb444c.png)

The other thing you can do here is enable insecure mode for the Traefik WebUI on port :8080 to see how it is configured.

```
      # Uncomment this to enable the Traefik WebUI on port :8080
      - "--api.insecure"
```

You can then tunnel with the Balena CLI as follows:

`balena tunnel your-fleet-name 8080:8080`

Use a browser to go to [localhost:8080](http://localhost:8080) and you'll see the Traefik WebUI

![image](https://user-images.githubusercontent.com/1537834/157541959-fd4c22a7-961e-4e51-aa90-6f4b65272c59.png)

# Rebuilding the Docker image and pushing to your own Docker registry

There is a [build-images.sh](https://github.com/DynamicDevices/reverse-proxy/blob/main/build-images.sh) script which I modified from the `pulse` block.

You'll need to change the Docker repo to your own and you should then be able to use this to build and upload the image(s).

I had problems as my installation of `docker buildx` didn't have support for ARM64 building and I had to look through the information [here](https://community.arm.com/arm-community-blogs/b/tools-software-ides-blog/posts/getting-started-with-docker-for-arm-on-linux)

Specifically I needed to register Arm executables to run on x64 machines:

```
docker run --rm --privileged docker/binfmt:820fdd95a9972a5308930a2bdfb8573dd4447ad3
```

To verify the qemu handlers are registered properly run:

```
cat /proc/sys/fs/binfmt_misc/qemu-aarch64
```

