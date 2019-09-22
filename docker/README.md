# Docker Image for FSCrawler

## Quick Reference

- Documentation

    https://fscrawler.readthedocs.io/en/latest/index.html

- Source Code

    [dadoonet/fscrawler](https://github.com/dadoonet/fscrawler)

## How to Run

```sh
$ docker run -it --rm -v ${PWD}/config:/root/.fscrawler -v ${PWD}/data:/tmp/es:ro toto1310/fscrawler fscrawler job_name
```

## Using with Elasticsearch installed by Docker

In case you also [install Elasticsearch with docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html), you should be able to connect each containers.

To connect to a docker container named `elasticsearch`, modify your `settings.yml` or `settings.json`.

```yml
elasticsearch:
  nodes:
  - url: "http://elasticsearch:9200"
```

### Legacy `--link` flag:

```sh
$ docker run --rm --name elasticsearch -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.3.2
```

```sh
$ docker run -it --rm --link elasticsearch -v ${PWD}/config:/root/.fscrawler -v ${PWD}/data:/tmp/es:ro toto1310/fscrawler fscrawler job_name
```

### Instead of above, use docker network:

```yml
version: '2.2'
services:
  # FSCrawler 
  fscrawler:
    image: toto1310/fscrawler
    container_name: fscrawler
    volumes:
      - ${PWD}/config:/root/.fscrawler
      - ${PWD}/data:/tmp/es
    networks: 
      - esnet
    command: fscrawler job_name
    
  # Elasticsearch Cluster
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.3.2
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - discovery.seed_hosts=elasticsearch2
      - cluster.initial_master_nodes=elasticsearch,elasticsearch2
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.3.2
    container_name: elasticsearch2
    environment:
      - node.name=elasticsearch2
      - discovery.seed_hosts=elasticsearch
      - cluster.initial_master_nodes=elasticsearch,elasticsearch2
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata02:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata01:
    driver: local
  esdata02:
    driver: local

networks:
  esnet:
```

```sh
$ docker-compose up -d elasticsearch elasticsearch2
```

```sh
$ docker-compose up fscrawler
```

## Running as non-root

By default, FSCrawler needs the user home(i.e. `~/.fscrawler` ) as a configuration directory, so you need to change it using `--config_dir` option.

```sh
$ docker run -it --rm --link elasticsearch -u 1000 -v ${PWD}/config:/tmp/config -v ${PWD}/data:/tmp/es:ro toto1310/fscrawler fscrawler --config_dir /tmp/config job_name
```
