---
title:  "Filter filebeat logs !"
date:   2022-01-24 11:00:00 +0530
categories: filebeat
author: Prabhat Kumar
header:
  image: /assets/images/bhemeswari.jpg
  image_description: "Bhemeswari"
---
# Sending selective Docker container Logs

## Requirement
There are multiple docker containers are running on system. In order not to write unnecessary information to the indexes, it require to send logs of selective containers.

Here is sample **docker-compose.yaml** config:

```
version: "3"
services:
	filebeat:
        image: "docker.elastic.co/beats/filebeat:7.2.0"
        user: root
        volumes:
            - /WORKDIR/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
            - /var/lib/docker:/var/lib/docker:ro
            - /var/run/docker.sock:/var/run/docker.sock
			- /WORKDIR/logs:/var/log/filebeat
```


Here is sample sample **filebeat.yml** config.   Filebeat **processors** name **drop_event.when** used to filer container logs by image name.

```
filebeat.inputs:
- type: container
  paths:
    - '/var/lib/docker/containers/*/*.log'

processors:
- add_docker_metadata:
    host: "unix:///var/run/docker.sock"
- drop_event.when.not.contains:
    container.image.name: "docker.elastic.co/beats/filebeat"


- decode_json_fields:
    fields: ["message"]
    target: "json"
    overwrite_keys: true

output.file:
    path: "/var/log/filebeat"
    filename: filebeat
logging.json: true
logging.metrics.enabled: false
```


Reference:-

1. https://www.sarulabs.com/post/5/2019-08-12/sending-docker-logs-to-elasticsearch-and-kibana-with-filebeat.html
2. https://ezyforanykey.blogspot.com/2020/11/filebeat-exclude-kubernetes-namespace.html
3. https://www.elastic.co/guide/en/beats/filebeat/current/defining-processors.html#condition-contains
