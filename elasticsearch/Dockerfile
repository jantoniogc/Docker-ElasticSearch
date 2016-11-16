FROM  ubuntu:14.04

MAINTAINER Juan Antonio Glez Cano <jgonzalez@opencanarias.es>

RUN apt-get update

RUN apt-get install -y software-properties-common

RUN echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections \
    && add-apt-repository -y ppa:webupd8team/java

RUN apt-get update

RUN apt-get install -y oracle-java8-installer \
    && update-java-alternatives -s java-8-oracle \
    && apt-get install oracle-java8-set-default \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /var/cache/oracle-jdk8-installer \
    && echo "networkaddress.cache.ttl=60" >> /usr/lib/jvm/java-8-oracle/jre/lib/security/java.security \
    && apt-get clean && rm -rf /var/lib/apt/lists/*


ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

ENV ES_VERSION=5.0.0

ADD https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-$ES_VERSION.tar.gz /tmp/es.tgz
RUN cd /usr/share && \
  tar xf /tmp/es.tgz && \
  rm /tmp/es.tgz

EXPOSE 9200 9300

ENV ES_HOME=/usr/share/elasticsearch-$ES_VERSION \
    DEFAULT_ES_USER=elasticsearch \
    DISCOVER_TRANSPORT_IP=eth0 \
    DISCOVER_HTTP_IP=eth0

#RUN adduser -S -s /bin/sh $DEFAULT_ES_USER

RUN adduser --home=/opt/elasticsearch --disabled-password --gecos "" --shell=/bin/sh $DEFAULT_ES_USER

VOLUME ["/data","/conf"]

WORKDIR $ES_HOME

COPY start /start
COPY ./config/log4j2.properties $ES_HOME/config/
COPY ./config/elasticsearch.yml $ES_HOME/config/
CMD ["/start"]
