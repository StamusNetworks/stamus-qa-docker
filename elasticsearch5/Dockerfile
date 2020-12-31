FROM docker.elastic.co/elasticsearch/elasticsearch:5.6.16

RUN echo 'discovery.type: single-node' >> /usr/share/elasticsearch/config/elasticsearch.yml
RUN echo 'xpack.security.enabled: false' >> /usr/share/elasticsearch/config/elasticsearch.yml
