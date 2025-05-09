FROM ubuntu:bionic

#  Copyright (c) 2018, The OpenThread Authors.
#  All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without
#  modification, are permitted provided that the following conditions are met:
#  1. Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#  2. Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in the
#     documentation and/or other materials provided with the distribution.
#  3. Neither the name of the copyright holder nor the
#     names of its contributors may be used to endorse or promote products
#     derived from this software without specific prior written permission.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
#  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
#  IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
#  ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
#  LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
#  CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
#  SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
#  INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
#  CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
#  ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
#  POSSIBILITY OF SUCH DAMAGE.
#

#ARG BASE_IMAGE=ubuntu:bionic
#FROM ${BASE_IMAGE}

ARG INFRA_IF_NAME
ARG BORDER_ROUTING
ARG BACKBONE_ROUTER
ARG OT_BACKBONE_CI
ARG OTBR_OPTIONS
ARG DNS64
ARG NAT64
ARG NAT64_SERVICE
ARG NAT64_DYNAMIC_POOL
ARG REFERENCE_DEVICE
ARG RELEASE
ARG REST_API
ARG WEB_GUI
ARG MDNS
ARG FIREWALL

ENV INFRA_IF_NAME=${INFRA_IF_NAME:-eth0}
ENV BORDER_ROUTING=${BORDER_ROUTING:-1}
ENV BACKBONE_ROUTER=${BACKBONE_ROUTER:-1}
ENV OT_BACKBONE_CI=${OT_BACKBONE_CI:-0}
ENV OTBR_MDNS=${MDNS:-mDNSResponder}
ENV OTBR_OPTIONS=${OTBR_OPTIONS}
ENV DEBIAN_FRONTEND noninteractive
ENV PLATFORM debian
ENV REFERENCE_DEVICE=${REFERENCE_DEVICE:-0}
ENV RELEASE=${RELEASE:-1}
ENV NAT64=${NAT64:-1}
ENV NAT64_SERVICE=${NAT64_SERVICE:-openthread}
ENV NAT64_DYNAMIC_POOL=${NAT64_DYNAMIC_POOL:-192.168.255.0/24}
ENV DNS64=${DNS64}
ENV WEB_GUI=${WEB_GUI:-1}
ENV REST_API=${REST_API:-1}
ENV FIREWALL=${FIREWALL:-1}
ENV DOCKER 1

RUN env

# Required during build or run
ENV OTBR_DOCKER_REQS sudo python3

# Required during build, could be removed
ENV OTBR_DOCKER_DEPS git ca-certificates

# Required and installed during build (script/bootstrap), could be removed
ENV OTBR_BUILD_DEPS apt-utils build-essential psmisc ninja-build cmake wget ca-certificates \
  libreadline-dev libncurses-dev libcpputest-dev libdbus-1-dev libavahi-common-dev \
  libavahi-client-dev libboost-dev libboost-filesystem-dev libboost-system-dev \
  libnetfilter-queue-dev

# Required for OpenThread Backbone CI
ENV OTBR_OT_BACKBONE_CI_DEPS curl lcov wget build-essential python3-dbus python3-zeroconf

# Required and installed during build (script/bootstrap) when RELEASE=1, could be removed
ENV OTBR_NORELEASE_DEPS \
  cpputest-dev

RUN apt-get update \
  && apt-get install --no-install-recommends -y $OTBR_DOCKER_REQS $OTBR_DOCKER_DEPS \
  && ([ "${OT_BACKBONE_CI}" != "1" ] || apt-get install --no-install-recommends -y $OTBR_OT_BACKBONE_CI_DEPS) \
  && ln -fs /usr/share/zoneinfo/UTC /etc/localtime

# Use my fork which has otbr-web on backbone not tun i/f
#COPY ./script /app/script
WORKDIR /app
#RUN git clone https://github.com/DynamicDevices/ot-br-posix.git /app
#RUN git checkout f2993f7
RUN git clone https://github.com/openthread/ot-br-posix.git /app

# Patch OTBR so that REST API runs on 8080 in order that we can access it through public Balena URL
#RUN sed -i 's/8081/8080/' src/web/web-service/frontend/res/js/app.js
#RUN sed -i 's/8081/8080/' src/agent/main.cpp

RUN ./script/bootstrap
RUN ./script/setup

COPY wait-for-otbr.sh /app
COPY form-network.sh /app
COPY docker_entrypoint.sh /app

ENTRYPOINT ["/app/docker_entrypoint.sh"]

#RUN  mv /tmp/etc .

# CUSTOMISE
# Patch OTBR so it calls REST API at port 80 which allows Traefik to forward the request (!)
RUN sed -i 's/8081/80/' /usr/share/otbr-web/frontend/res/js/app.js

RUN apt-get -q update
RUN apt-get install -yq netcat
RUN apt-get install -yq mosquitto-clients

#RUN update-alternatives --set iptables /usr/sbin/iptables-legacy
#RUN update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
#RUN update-alternatives --set arptables /usr/sbin/arptables-legacy
#RUN update-alternatives --set ebtables /usr/sbin/ebtables-legacy

#
# --- BEGIN MQTT-SNGATEWAY
#
# TODO: We put MQTT-SNGateway in this container as I haven't worked out how to route mesh packets from wpan0 between containers
RUN apt update && apt -y install git cmake libbluetooth-dev

#RUN git clone https://github.com/eclipse/paho.mqtt-sn.embedded-c.git 
#RUN cd paho.mqtt-sn.embedded-c && git checkout 59797127e7f3d024de576555cd4232c68e874ac6

# AJL - Use my fork with UDP6 broadcast/multicast fix
WORKDIR /app
RUN git clone https://github.com/DynamicDevices/paho.mqtt-sn.embedded-c.git -b ajl/fix-debug-ipv6-addr-output
RUN cd paho.mqtt-sn.embedded-c && git checkout bd99ee2

COPY docker_entrypoint.sh /app
ENTRYPOINT ["/app/docker_entrypoint.sh"]

# Enable DEBUG output

#RUN sed -i 's|//#define DEBUG_MQTTSN |#define DEBUG_MQTTSN|' paho.mqtt-sn.embedded-c/MQTTSNGateway/src/MQTTSNGWDefines.h
#RUN sed -i 's|//#define DEBUG_NW |#define DEBUG_NW|' paho.mqtt-sn.embedded-c/MQTTSNGateway/src/MQTTSNGWDefines.h

RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && ./build.sh udp
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && cp ./bin/MQTT-SNGateway /app/MQTT-SNGateway.udp &&  cp ../build.gateway/MQTTSNPacket/src/libMQTTSNPacket.so /app/libMQTTSNPacket.so && cp ./bin/MQTT-SNLogmonitor /app/MQTT-SNLogmonitor.udp
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && ./build.sh udp6
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && cp ./bin/MQTT-SNGateway /app/MQTT-SNGateway.udp6 &&  cp ./bin/MQTT-SNLogmonitor /app/MQTT-SNLogmonitor.udp6
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && ./build.sh dtls
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && cp ./bin/MQTT-SNGateway /app/MQTT-SNGateway.dtls &&  cp ./bin/MQTT-SNLogmonitor /app/MQTT-SNLogmonitor.dtls
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && ./build.sh dtls6
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && cp ./bin/MQTT-SNGateway /app/MQTT-SNGateway.dtls6 &&  cp ./bin/MQTT-SNLogmonitor /app/MQTT-SNLogmonitor.dtls6
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && ./build.sh xbee
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && cp ./bin/MQTT-SNGateway /app/MQTT-SNGateway.xbee &&  cp ./bin/MQTT-SNLogmonitor /app/MQTT-SNLogmonitor.xbee
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && ./build.sh loralink
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && cp ./bin/MQTT-SNGateway /app/MQTT-SNGateway.loralink &&  cp ./bin/MQTT-SNLogmonitor /app/MQTT-SNLogmonitor.loralink
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && ./build.sh rfcomm
RUN cd paho.mqtt-sn.embedded-c/MQTTSNGateway && cp ./bin/MQTT-SNGateway /app/MQTT-SNGateway.rfcomm &&  cp ./bin/MQTT-SNLogmonitor /app/MQTT-SNLogmonitor.rfcomm

COPY wait-for-otbr.sh /app
COPY form-network.sh /app
COPY run-mqtt-sngateway.sh /app
COPY *.conf.in /app/

# Tidy up
RUN  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $OTBR_DOCKER_DEPS
RUN  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $OTBR_BUILD_DEPS
#RUN  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false "$OTBR_NORELEASE_DEPS"
RUN rm -rf /var/lib/apt/lists/*

# --- END MQTT-SNGATEWAY
