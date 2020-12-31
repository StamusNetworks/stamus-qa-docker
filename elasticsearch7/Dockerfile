FROM docker.elastic.co/elasticsearch/elasticsearch:7.10.1

RUN echo 'discovery.type: single-node' >> /usr/share/elasticsearch/config/elasticsearch.yml
RUN echo 'xpack.security.enabled: false' >> /usr/share/elasticsearch/config/elasticsearch.yml
