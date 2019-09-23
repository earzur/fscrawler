FROM maven:3.3-jdk-8 AS build

### Download and cache the dependency jar
WORKDIR /root/fscrawler 
COPY ./pom.xml /root/fscrawler/pom.xml
COPY ./beans/pom.xml /root/fscrawler/beans/pom.xml
COPY ./cli/pom.xml /root/fscrawler/cli/pom.xml
COPY ./core/pom.xml /root/fscrawler/core/pom.xml
COPY ./crawler/pom.xml /root/fscrawler/crawler/pom.xml
COPY ./crawler/crawler-abstract/pom.xml /root/fscrawler/crawler/crawler-abstract/pom.xml
COPY ./crawler/crawler-ssh/pom.xml /root/fscrawler/crawler/crawler-ssh/pom.xml
COPY ./crawler/crawler-fs/pom.xml /root/fscrawler/crawler/crawler-fs/pom.xml
COPY ./distribution/pom.xml /root/fscrawler/distribution/pom.xml
COPY ./distribution/es5/pom.xml /root/fscrawler/distribution/es5/pom.xml
COPY ./distribution/es6/pom.xml /root/fscrawler/distribution/es6/pom.xml
COPY ./elasticsearch-client/pom.xml /root/fscrawler/elasticsearch-client/pom.xml
COPY ./elasticsearch-client/elasticsearch-client-base/pom.xml /root/fscrawler/elasticsearch-client/elasticsearch-client-base/pom.xml
COPY ./elasticsearch-client/elasticsearch-client-v5/pom.xml /root/fscrawler/elasticsearch-client/elasticsearch-client-v5/pom.xml
COPY ./elasticsearch-client/elasticsearch-client-v6/pom.xml /root/fscrawler/elasticsearch-client/elasticsearch-client-v6/pom.xml
COPY ./framework/pom.xml /root/fscrawler/framework/pom.xml
COPY ./integration-tests/pom.xml /root/fscrawler/integration-tests/pom.xml
COPY ./integration-tests/it-common/pom.xml /root/fscrawler/integration-tests/it-common/pom.xml
COPY ./integration-tests/it-v5/pom.xml /root/fscrawler/integration-tests/it-v5/pom.xml
COPY ./integration-tests/it-v6/pom.xml /root/fscrawler/integration-tests/it-v6/pom.xml
COPY ./rest/pom.xml /root/fscrawler/rest/pom.xml
COPY ./settings/pom.xml /root/fscrawler/settings/pom.xml
COPY ./test-documents/pom.xml /root/fscrawler/test-documents/pom.xml
COPY ./test-framework/pom.xml /root/fscrawler/test-framework/pom.xml
COPY ./tika/pom.xml /root/fscrawler/tika/pom.xml
RUN set -ex \
    && mvn -B -s /usr/share/maven/ref/settings-docker.xml clean dependency:resolve dependency:resolve-plugins

### Build and Package into zip
COPY . /root/fscrawler
RUN set -ex \
    && mvn -B -s /usr/share/maven/ref/settings-docker.xml package -DskipTests
RUN set -ex \
    && unzip /root/fscrawler/distribution/es6/target/*.zip -d /root/fscrawler_tmp && mv /root/fscrawler_tmp/* /root/fscrawler_es6 \
    && unzip /root/fscrawler/distribution/es5/target/*.zip -d /root/fscrawler_tmp && mv /root/fscrawler_tmp/* /root/fscrawler_es5
