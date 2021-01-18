---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "ELK"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2021-01-18T11:33:05+08:00
lastmod: 2021-01-18T11:33:05+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

ELK

### Docker Compose方式部署

采集docker容器日志

#### 目录结构

```
.
├── docker-compose.yml
├── filebeat.yml
└── pipeline
    └── logstash.conf

1 directory, 3 files
```

#### docker-compose.yml

```
version: '3.3'
services:

  elasticsearch:
    hostname: elasticsearch
    image: docker.elastic.co/elasticsearch/elasticsearch:$ELASTIC_VERSION
    environment:
      - TZ=Asia/Shanghai
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type=single-node
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - stack

  kibana:
    hostname: kibana
    image: docker.elastic.co/kibana/kibana:$ELASTIC_VERSION
    environment:
      - TZ=Asia/Shanghai
    depends_on:
      - elasticsearch
    ports:
      - 5601:5601
    networks:
      - stack

  logstash:
    hostname: logstash
    image: docker.elastic.co/logstash/logstash:$ELASTIC_VERSION
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./pipeline/:/usr/share/logstash/pipeline/
    ports:
      - 5044:5044
    depends_on:
      - elasticsearch
    networks:
      - stack

  filebeat:
    hostname: filebeat
    image: docker.elastic.co/beats/filebeat:${ELASTIC_VERSION}
    environment:
      - TZ=Asia/Shanghai
    user: root
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    command: filebeat -e
    depends_on:
      - elasticsearch
      - kibana
    restart: on-failure
    networks:
      - stack

volumes:
  esdata1:
    driver: local
networks:
  stack:
```

#### filebeat.yml

```
filebeat.autodiscover:
  providers:
    - type: docker
      templates:
        - condition:
            contains:
              docker.container.labels.com.docker.compose.project.working_dir: metasearch
          config:
            - type: container
              paths:
                - /var/lib/docker/containers/${data.docker.container.id}/*.log
              multiline.pattern: '^([0-9]{4}-[0-9]{2}-[0-9]{2})'
              multiline.negate: true
              multiline.match: after

output.logstash:
  hosts: ["logstash:5044"]
```

#### logstash.conf

```
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => {
      "message" => [
        "(?m)%{TIMESTAMP_ISO8601:timestamp}%{SPACE}%{LOGLEVEL:level} %{NUMBER} --- \[\s*%{DATA:thread}\] [A-Za-z0-9.]*\.(?<class>[A-Za-z0-9#_$]+)\s*:\s+%{GREEDYDATA:logmessage}",
        "(?m)%{TIMESTAMP_ISO8601:timestamp}%{SPACE}%{LOGLEVEL:level} \[%{DATA:thread}\][A-Za-z0-9.]*\.(?<class>[A-Za-z0-9#_$]+)\.(?<method>[A-Za-z0-9#_$<>]+):%{NUMBER:line} -%{GREEDYDATA:logmessage}"
      ]
    }
  }
  date {
    match => ["timestamp", "ISO8601", "yyyy-MM-dd HH:mm:ss.SSS"]
  }
  mutate {
    remove_field => ["docker", "container", "agent", "ecs", "stream"]
    add_field => {
      "service" => "%{[docker][container][labels][com_docker_compose_service]}"
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "metasearch-log-%{+YYYY.MM.dd}" 
  }
}
```

