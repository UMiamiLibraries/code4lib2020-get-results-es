---
layout: page
title: Creating the Dockerfile
permalink: /creating-dockerfile/
nav_order: 3
---

## Creating the Dockerfile

<p>A Dockerfile contains all the commands needed to build a specific image. Think of a Dockerfile as a recipe 
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

<p>Now, let's install php composer. Add the following code to the Dockerfile.</p>

<p>
{% highlight Docker %}
#.docker/Dockerfile

# Installing php composer
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

### Adding an ENTRYPOINT to the Dockerfile

<p>
An ENTRYPOINT will let us specify which commands we want to execute when the container is started. Let's, for now, add the following code to our Dockerfile
</p>

<p>
{% highlight Docker %}
#.docker/Dockerfile

COPY ./.docker/init.sh /usr/local/bin/init.sh
RUN chmod u+x /usr/local/bin/init.sh

ENTRYPOINT ["/usr/local/bin/init.sh"]

{% endhighlight %} 
</p>

<p>
The previous code will copy the init.sh file into the Docker container and define an ENTRYPOINT using the init.sh
But wait, the init.sh file does not exist. Let's create an init.sh file in the .docker directory and add the following code to it:
</p>

<p>
{% highlight bash %}
#!/usr/bin/env bash

echo "Checking Elasticsearch node 1 is ready"
/usr/local/wait-for-it.sh esdata-0.local:9200 -s --timeout=120 -- echo "Node 1 is ready!"

echo "Checking Elasticsearch node 2 is ready"
/usr/local/wait-for-it.sh esdata-1.local:9200 -s --timeout=120 -- echo "Node 2 is ready!"

# Install search app dependencies
echo "Installing search app dependencies"
cd /home/site/wwwroot
composer install

# start apache
/usr/sbin/apache2ctl -D FOREGROUND

{% endhighlight %} 
</p>

<p>This code will wait 2 minutes for the Elasticsearch containers (we will create these in a later section) to be ready. It will also install the search app dependencies defined in the composer.json file, and then starts up the Apache server.</p>

<p>The Dockerfile should look like this at this point:</p>

<p>
{% highlight docker %}

FROM appsvc/php:7.3-apache_20200101.1

# Installing php composer
RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer && \
        chmod +x /usr/local/bin/composer && \
        composer --version

#Remove php opcache file -- only for development
RUN rm -rf /usr/local/etc/php/conf.d/opcache-recommended.ini

COPY ./.docker/wait-for-it/wait-for-it.sh /usr/local/wait-for-it.sh
RUN chmod u+x /usr/local/wait-for-it.sh

COPY ./.docker/init.sh /usr/local/bin/init.sh
RUN chmod u+x /usr/local/bin/init.sh

ENTRYPOINT ["/usr/local/bin/init.sh"]


{% endhighlight %} 
</p>

<p>Time to build the container. Run the following code from the project root folder in a console:</p>

<p>
{% highlight console %}

$ docker build -f .docker/Dockerfile -t search-app:v1.0.0 .

{% endhighlight %} 
</p>

<<<<<<< HEAD
<p>The previous command tells Docker to build a container using the Dockerfile defined in the -f parameter. I
f you don't specify a filepath, Docker will search for a Dockefile within the same directory the build command is executed from.
The dot at the end of the command specifies the location where we want to build the container. In this case, we are building
the container in the current directory</p>
=======
<p>The previous command tells Docker to build a container using the Dockerfile defined in the -f parameter. 
If you don't specify a filepath, Docker will search for a Dockefile within the same directory the build command is executed from.
We are also tagging the Docker container by using the -t parameter. The dot at the end of the command specified the 
location where we want to build the container. In this case, we are building
the container in the current directory.</p>
>>>>>>> afc-dev

<p>The output from the previous docker build command should look similar to this: </p>

<p>
{% highlight console %}
Sending build context to Docker daemon  286.2kB
Step 1/7 : FROM appsvc/php:7.2-apache_20191031.7
 ---> 692faef99277
Step 2/7 : RUN curl -sS https://getcomposer.org/installer | php
 ---> Running in 59de52d7babe
Cannot load Zend OPcache - it was already loaded
All settings correct for using Composer
Downloading...

Composer (version 1.9.3) successfully installed to: /home/site/wwwroot/composer.phar
Use it: php composer.phar

Removing intermediate container 59de52d7babe
 ---> 11c9d3ca3665
Step 3/7 : RUN mv composer.phar /usr/local/bin/composer && chmod +x /usr/local/bin/composer && composer --version
 ---> Running in ea691c6d9ec2
Cannot load Zend OPcache - it was already loaded
Composer version 1.9.3 2020-02-04 12:58:49
Removing intermediate container ea691c6d9ec2
 ---> 220bcb9bfeb3
Step 4/7 : RUN rm -rf /usr/local/etc/php/conf.d/opcache-recommended.ini
 ---> Running in 5b0c82733c56
Removing intermediate container 5b0c82733c56
 ---> 8bb2be6b4690
Step 5/7 : COPY ./.docker/init.sh /usr/local/bin/init.sh
 ---> 51ac59253e84
Step 6/7 : RUN chmod u+x /usr/local/bin/init.sh
 ---> Running in 78fd768bc949
Removing intermediate container 78fd768bc949
 ---> f0db0655f565
Step 7/7 : ENTRYPOINT ["/usr/local/bin/init.sh"]
 ---> Running in 5d6b187e36c2
Removing intermediate container 5d6b187e36c2
 ---> 67a994a34f76
Successfully built 67a994a34f76
{% endhighlight %} 
</p>

<p>
So far we have written a Dockerfile to build a Docker container running PHP, Apache, 
php-composer and a custom ENTRYPOINT. 
In the next sections, we will create a docker-compose file to configure other containers that we need for our search application.
</p>

<hr>

[Previous: Project Setup]({{ site.baseurl }}/project-setup){: .btn .btn-outline }
[Next: The docker-compose file]({{ site.baseurl }}/docker-compose-file/){: .btn .btn-outline }