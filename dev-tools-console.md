---
layout: page
title: The Dev Tools Console
permalink: /dev-tools-console/
nav_order: 8
---

## The Dev Tools Console

<p>Kibana is an open source data visualization dashboard for Elasticsearch. The Dev Tools Console is one of the many
tools that Kibana offers to manage your Elasticsearch data.</p>

<p>First, let's add a Kibana service to our docker-compose file. Be sure to stop docker-compose by pressing Ctrl+C in the console
where you are running docker-compose or run 'docker-compose stop' from a new console.</p>

<p>Add the following code to the services section of the docker-compose file:</p>

<p>
{% highlight YAML %}
  kibana:
    image: docker.elastic.co/kibana/kibana:7.6.1
    environment:
      ELASTICSEARCH_HOSTS: http://esdata-0.local:9200
    ports:
      - 5601:5601 # Exposing port 5601 of the container
{% endhighlight %} 
</p>

<p>We are defining a kibana service using the official Kibana Docker image. We are also defining the esdata-0.local node
 as the host that Kibana will connect to. The port 5601 is exposed to access Kibana from our browser.</p>

<p>Run docker-compose up from a console. This operation can take a few minutes depending on the host computer hardware.</p>

<p>
{% highlight console %}
$ docker-compose up
{% endhighlight %} 
</p>

<p>When all of the services are running, open http://localhost:5601/ in your browser. You should see the Kibana homepage:</p>

<figure>
  <img src="{{ site.baseurl }}/assets/images/kibana-homepage.jpg" alt="Image displays the Kibana homepage">
</figure>

<p>Click the Console link under Manage and Administer the Elastic Stack</p>

<figure>
  <img src="{{ site.baseurl }}/assets/images/kibana-homepage-console-highlitghted.JPG" alt="Image displays the Kibana homepage with console link highlighted">
</figure>

<p>You should see the Dev Tools</p>

<figure>
  <img src="{{ site.baseurl }}/assets/images/kibana-dev-tools.JPG" alt="Image displays the Kibana Dev Tools">
</figure>

<p>You can use the console to interact with the data in your cluster directly.</p>
<hr>

[Previous: Searching multiple indexes]({{ site.baseurl }}/searching-multiple-indexes/){: .btn .btn-outline }
[Next: Useful links]({{ site.baseurl }}/useful-links/){: .btn .btn-outline }