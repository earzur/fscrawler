FROM toto1310/fscrawler:2.7-SNAPSHOT-build as build

FROM openjdk:8-jdk AS runtime

ARG es=es7
COPY --from=build /root/fscrawler_${es} /usr/share/fscrawler
RUN set -ex \
    && ln -sn /usr/share/fscrawler/bin/fscrawler /usr/bin/

WORKDIR /usr/share/fscrawler
