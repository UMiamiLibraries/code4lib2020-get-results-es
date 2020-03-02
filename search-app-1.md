---
layout: page
title: The search app
permalink: /search-app-1/
nav_order: 5
---

## The search app

<p>
We are now ready to create our docker-compose file. Our development environment will consist of two PHP applications and 
a two-node Elasticsearch cluster. Let's get started.
</p>

<p>
Let's start by defining the two-node Elasticsearch cluster. Add the following code to the docker-compose.yml file at the root of the project directory. Keep in mind this is a YAML file,
so indentation matters.
</p>

<p>
{% highlight YAML %}
# docker-compose.yml
services:
  esdata-0.local: # First Elasticsearch node
    container_name: esdata-0.local
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.0 # Official Elasticsearch image
    environment: # Specific container environment variables
      - node.name=esdata-0.local # Name of the Elasticsearch node
      - cluster.name=es-local-cluster # Name of the Elasticsearch cluster
      - discovery.seed_hosts=esdata-1.local #List of other nodes in the cluster that are likely to be live and contactable
      - cluster.initial_master_nodes=esdata-0.local # List of other nodes in the cluster that are master eligible
      - bootstrap.memory_lock=true # Prevents any Elasticsearch object in memory from being swapped out to the hard drive
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" # Sets JVM heap size in the container
    volumes: # Attach local data directory
      - esdata-0.local.data:/usr/share/elasticsearch/data # Docker volume to persist the node data across restarts
    ports:
      - 9200:9200 # Exposing port 9200 of the container
    ulimits:
      memlock: # Sets an unlimited amount of memory to be locked by the service (container)
        soft: -1
        hard: -1

  esdata-1.local: # Second Elasticsearch node
    container_name: esdata-1.local
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.0
    environment:
      - node.name=esdata-1.local
      - cluster.name=es-local-cluster
      - discovery.seed_hosts=esdata-0.local
      - cluster.initial_master_nodes=esdata-0.local
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - esdata-1.local.data:/usr/share/elasticsearch/data
    ulimits:
      memlock:
        soft: -1
        hard: -1

volumes: # Defines volumes used by the services
  esdata-0.local.data:
    driver: local
  esdata-1.local.data:
    driver: local

{% endhighlight %} 
</p>

<p>
Let's run our cluster with Docker-Compose. Run the following command in a console in the same directory where
the docker-compose file is located.
</p>

<p>
{% highlight console %}
$ docker-compose up
{% endhighlight %} 
</p>

<p>
Docker-Compose will pull the Elasticsearch v7.6.0 image and start the two services we defined. This operation can take 
a few minutes depending on the host computer hardware.
</p>

<p>
Let's check if the cluster is running. Open a browser and visit the URL http://localhost:9200/_nodes/stats
</p>

<p>
We are querying the Nodes stats API of Elasticsearch. The response should look similar to this (nodes details collapsed on purpose):
</p>

<p>
{% highlight JSON %}
{
    "_nodes": {
        "total": 2,
        "successful": 2,
        "failed": 0
    },
    "cluster_name": "es-local-cluster",
    "nodes": {
        "BkODnEDlS4mjSh4tg6NeTA": {}, // 20 items
        "TTTdcluiRrSao-9vFdSpvQ": {} // 20 items
    }
}
{% endhighlight %} 
</p>

<p>
Go back to the console from where you ran the docker-compose up command and press Ctrl+C to stop the containers.
</p>

<p>
So far we have defined a two-node Elasticsearch cluster. These two nodes (containers) are using the official image 
of Elasticsearch v7.6.0 and are part of the es-local-cluster cluster. We also defined two volumes to be used by each node,
so the node data is persisted across restarts of the container.  
</p>

### Adding a service for our custom app
<p>
Now we have a working Elasticsearch cluster. It is time to define the other service that will run the custom app
we are going to create.
</p>

<p>
Add the below service in the docker-compose file, after the second Elasticsearch service.
</p>

<p>
{% highlight YAML %}
# docker-compose.yml

  search-app: # Search App service
    container_name: search-app
    build:
      context: ./
      dockerfile: ./.docker/Dockerfile
    environment: # Specific container environment variables
      - APACHE_PORT=80 # Set Apache to listen on port 80
      - APACHE_DOCUMENT_ROOT=/home/site/wwwroot/public #Set the directory from which Apache will serve files
    volumes: # Attach local data directory
      - ./search-app:/home/site/wwwroot
    depends_on:
      - esdata-0.local
      - esdata-1.local
    ports:
      - 8080:80

{% endhighlight %} 
</p>

<p>Run docker-compose up again and visit http://localhost:8080/public/ in your browser. You should get the following: </p>

<p>
{% highlight console %}
$ docker-compose up
{% endhighlight %} 
</p>

<p>
<figure>
  <img src="{{ site.baseurl }}/assets/images/search-app-running-from-docker-container.png" alt="Image displays Search app running on the Docker Container">
</figure>
</p>

<p>We now have in place all the infrastructure for our search app. Let's create the search app!</p>

<hr>

[Previous: The docker-compose file]({{ site.baseurl }}/docker-compose-file){: .btn .btn-outline }
[Next: TBD]({{ site.baseurl }}/creating-dockerfile/){: .btn .btn-outline }