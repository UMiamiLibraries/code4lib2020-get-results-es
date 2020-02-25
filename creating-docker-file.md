---
layout: page
title: Creating the Dockerfile
permalink: /creating-dockerfile/
nav_order: 3
---

## Creating the Dockerfile

<p>A Dockerfile contains all the commands needed to build a specific image. Think off a Dockerfile as a recipe 
to build an image.</p>

<p>Docker images are formed by multiple read-only layers that are representations of the instructions contained 
in the Dockerfile. Each layer builds on top of the previous layer.</p>

<p>Let's create a new file named Dockerfile in the .docker directory and add the following code to it.</p>

<p>
{% highlight Docker %}
#.docker/Dockerfile

FROM appsvc/php:7.3-apache_20200101.1

{% endhighlight %} 
</p>

<p>We are instructing Docker to start FROM the pre-existing PHP and Apache image built by Microsoft, appsvc/php:7.3-apache_20200101.1. 
This pre-existing image is running a Linux distribution and php version 7.3 and apache, and it was created on January 1st, 2020.</p>

<p>Now, let's install composer. Add the following code to the Dockerfile.</p>

<p>
{% highlight Docker %}
#.docker/Dockerfile

RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer && \
        chmod +x /usr/local/bin/composer && \
        composer --version

{% endhighlight %} 
</p>

<p>The pre-existing image that we are using comes with PHP Opcache enabled. 
This extension helps to improve PHP performance by caching scripts in memory. 
However, since we are going to be using this container for development purposes, we want to disable caching for now.
In order to do that, let's insert the following code into our Dockerfile.</p>

<p>
{% highlight Docker %}
#.docker/Dockerfile

#Remove php opcache file -- only for development
RUN rm -rf /usr/local/etc/php/conf.d/opcache-recommended.ini

{% endhighlight %} 
</p>

### Synchronizing spin-up of independent containers

<p>
In this workshop we are going to have multiple containers running at the same time.
However, we might run into a scenario where our development container depends on another container to work, 
and not only another container, but also the services running inside this other container to be up and running.
</p>

<p>
To address this problem, we are going to use <a href="https://github.com/vishnubob/wait-for-it">wait-for-it</a>, a bash
script that will make our development container wait for any other container it depends on to continue. Add the following code to your Dockerfile:
</p>

<p>
{% highlight Docker %}
#.docker/Dockerfile

COPY ./wait-for-it/wait-for-it.sh /usr/local/wait-for-it.sh
RUN chmod u+x /usr/local/wait-for-it.sh

{% endhighlight %} 
</p>

<p>
We also need to tell Docker that the container is going to be listening on a specific port at runtime. Please add the following code to the Dockerfile:
</p>

<p>
{% highlight Docker %}
#.docker/Dockerfile

EXPOSE 80

{% endhighlight %} 
</p>

<p>Our container will be listening to request on port 80</p>

### Adding an ENTRYPOINT to the Dockerfile

<p>
An ENTRYPOINT will let us specify which commands we want to execute when the container is started. Let's, for now, add the following code to our Dockerfile
</p>

<p>
{% highlight Docker %}
#.docker/Dockerfile

COPY ./init.sh /usr/local/bin/init.sh
RUN chmod u+x /usr/local/bin/init.sh

ENTRYPOINT ["/usr/local/bin/init.sh"]

{% endhighlight %} 
</p>

<p>
But wait, the init.sh file does not exist. Let's create an init.sh file in the .docker directory and add the following code to it:
</p>

<p>
{% highlight console %}
#!/usr/bin/env bash

echo "Checking Elasticsearch node 1 is ready"
/usr/local/wait-for-it.sh umle1data-0.local:9200 -s --timeout=60 -- echo "Elasticsearch node 1 is ready!"

echo "Checking Elasticsearch node 2 is ready"
/usr/local/wait-for-it.sh umle1data-1.local:9200 -s --timeout=60 -- echo "Elasticsearch node 2 is ready!"

# start apache
sed -i "s/{PORT}/80/g" /etc/apache2/apache2.conf
/usr/sbin/apache2ctl -D FOREGROUND

{% endhighlight %} 
</p>

<p>This code will wait for the Elasticsearch containers (we will create these in later sections) to be ready and then starts up the Apache server listening on port 80 too.</p>

[Previous: Project Setup]({{ site.baseurl }}/project-setup){: .btn .btn-outline }
[Next: TBD]({{ site.baseurl }}/creating-dockerfile/){: .btn .btn-outline }