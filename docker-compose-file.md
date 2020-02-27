---
layout: page
title: The docker-compose file
permalink: /docker-compose-file/
nav_order: 4
---

## The docker-compose file

<p>
We are now ready to create our docker-compose file. Our development environment will consist of two PHP applications and 
a two-node Elasticsearch cluster. Let's get started.
</p>

<p>
We will beging by defining the Elasticsearch cluster. Add the following code to the docker-compose.yml file at the root of the project directory. Keep in mind this is a YAML file,
so indentation matters.
</p>

{% highlight YAML %}
# docker-compose.yml

services:

  esdata-0.local: # First Elasticsearch node
    container_name: esdata-0.local
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.0
    environment:
      - node.name=esdata-0.local
      - cluster.name=es-local-cluster
      - cluster.initial_master_nodes=esdata-0.local,esdata-1.local
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes: # Attach local data directory
      - esdata-0.local.data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    ulimits:
      memlock:
        soft: -1
        hard: -1
    networks:
      elastic-local-network:
        ipv4_address: 10.5.0.2


  esdata-1.local: # First Elasticsearch node
    container_name: esdata-1.local
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.0
    environment:
      - node.name=esdata-1.local
      - cluster.name=es-local-cluster
      - cluster.initial_master_nodes=esdata-0.local,esdata-1.local
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes: # Attach local data directory
      - esdata-1.local.data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    ulimits:
      memlock:
        soft: -1
        hard: -1
    networks:
      elastic-local-network:
        ipv4_address: 10.5.0.3

{% endhighlight %} 

<hr>

[Previous: Creating the Dockerfile]({{ site.baseurl }}/creating-dockerfile){: .btn .btn-outline }
[Next: TBD]({{ site.baseurl }}/creating-dockerfile/){: .btn .btn-outline }