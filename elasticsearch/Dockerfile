FROM docker.elastic.co/elasticsearch/elasticsearch:6.5.4

RUN echo 'discovery.type: single-node' >> /usr/share/elasticsearch/config/elasticsearch.yml
RUN echo 'xpack.security.enabled: false' >> /usr/share/elasticsearch/config/elasticsearch.yml
