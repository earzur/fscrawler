FROM toto1310/fscrawler:2.7-SNAPSHOT-build as build

FROM openjdk:8-jdk AS runtime

RUN set -ex \
    && apt-get update \
    && apt-get install -y \
        gettext-base \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ARG es=es6
COPY --from=build /root/fscrawler_${es} /usr/share/fscrawler
RUN set -ex \
    && ln -sn /usr/share/fscrawler/bin/fscrawler /usr/bin/

WORKDIR /usr/share/fscrawler
