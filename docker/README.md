# Docker for FSCrawler

Info: Unfortunately, this repository is still experimental. As a result, changes that are not backward compatible might occur unexpectedly. Also, do not create a new image based on this image unless it is very necessary.

## Quick Reference

- Documentation

    https://fscrawler.readthedocs.io/en/latest/index.html

- Source Code

    [dadoonet/fscrawler](https://github.com/dadoonet/fscrawler)

- Dockerfile(s)

    [See last section for details](#tags-and-respective-git-branch-links)

## How to Run

Refer to [Getting Started](https://fscrawler.readthedocs.io/en/latest/user/getting_started.html), you can run FSCrawler that read its configuration files from `/root/.fscrawler`(i.e. `--config_dir`) and its target files from `/tmp/es`(i.e. `fs.url`).

```sh
$ docker run -it --rm -v ${PWD}/config:/root/.fscrawler -v ${PWD}/data:/tmp/es:ro toto1310/fscrawler fscrawler job_name
```

## Using with Elasticsearch installed by Docker

If you [installing Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html), you need to communicate between each container.

For example, to connect to a docker container named `elasticsearch`, modify your `settings.yml` or `settings.json`.

```yml
elasticsearch:
  nodes:
  - url: "http://elasticsearch:9200"
```

After that, you have two ideas to do so. 

### As legacy way, use `--link` flag:

WARNING: That is easy, but DEPRECATED feature(Docker v19.03). For more details, please read [Legacy container links | Docker Documentation](https://docs.docker.com/network/links/).

For example, you run Elasticsearch in a container named "elasticsearch" as follows:

```sh
$ docker run --rm --name elasticsearch -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.3.2
```

Then, you can run FSCrawler as follows:

```sh
$ docker run -it --rm --link elasticsearch -v ${PWD}/config:/root/.fscrawler -v ${PWD}/data:/tmp/es:ro toto1310/fscrawler fscrawler job_name
```

### Instead of above, use docker network:

For example, prepare the following `docker-compose.yml`.

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

Then, you can run Elasticsearch.

```sh
$ docker-compose up -d elasticsearch elasticsearch2
```

After starting Elasticsearch, you can run FSCrawler.

```sh
$ docker-compose up fscrawler
```

## Using environment variables in FSCrawler configuration

Out-of-the-box, FSCrawler doesn't support environment variables inside  FSCrawler configuration such as `_settings.yml` or `_settings.json`. But `envsubst` may be used as a workaround if you need to generate your FSCrawler configuration dynamically before FSCrawler starts.

Here is an example using `docker-compose.yml`:

```yml
version: '3'
services:
  # FSCrawler 
  fscrawler:
    image: toto1310/fscrawler
    container_name: fscrawler
    volumes:
      - ${PWD}/config:/root/.fscrawler
      - ${PWD}/data:/tmp/es
    environment:
      - FS_URL=/tmp/es
      - ELASTICSEARCH_URL=http://elasticsearch:9200
      - ELASTICSEARCH_INDEX=job_index
    command: |
      /bin/bash -c "
        envsubst < /root/.fscrawler/_template.yml > /root/.fscrawler/_settings.yml \
        && fscrawler job_name"
```

The `_template.json` file may then contain variable references like this:

```yml
fs:
  url: "${FS_URL}"
```

## Running as non-root

By default, FSCrawler needs the user home(exactly `~/.fscrawler` ) as a configuration directory but not exist, so you need to change it using the` --config-dir` option in addition to docker's `-u` option.

```sh
$ docker run -it --rm -u 1000 -v ${PWD}/config:/tmp/config -v ${PWD}/data:/tmp/es:ro toto1310/fscrawler fscrawler --config_dir /tmp/config job_name
```

## Tags and Respective git branch links

- 2.7-SNAPSHOT(current)
    - [`latest`, `2.7-SNAPSHOT`, `2.7-SNAPSHOT-es7-eng`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
    - [`2.7-SNAPSHOT-build`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
    - For Elasticsearch 7
      - [`2.7-SNAPSHOT-es7-nolang`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
      - [`2.7-SNAPSHOT-es7-fra`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
      - [`2.7-SNAPSHOT-es7-jpn`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
    - For Elasticsearch 6
      - [`2.7-SNAPSHOT-es6-nolang`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
      - [`2.7-SNAPSHOT-es6-eng`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
      - [`2.7-SNAPSHOT-es6-fra`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
      - [`2.7-SNAPSHOT-es6-jpn`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
    - For Elasticsearch 5
      - [`2.7-SNAPSHOT-es5-nolang`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
      - [`2.7-SNAPSHOT-es5-eng`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
      - [`2.7-SNAPSHOT-es5-fra`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
      - [`2.7-SNAPSHOT-es5-jpn`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.7/docker)
- 2.6
    - [`2.6`, `2.6-es6-eng`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
    - [`2.6-build`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
    - For Elasticsearch 6
      - [`2.6-es6-nolang`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
      - [`2.6-es6-fra`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
      - [`2.6-es6-jpn`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
    - For Elasticsearch 5
      - [`2.6-es5-nolang`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
      - [`2.6-es5-eng`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
      - [`2.6-es5-fra`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
      - [`2.6-es5-jpn`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.6/docker)
- 2.5
    - [`2.5`, `2.5-eng`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.5/docker)
    - [`2.5-es6-nolang`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.5/docker)
    - [`2.5-es6-fra`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.5/docker)
    - [`2.5-es6-jpn`](https://github.com/toto1310/fscrawler/tree/dockerfile-2.5/docker)
