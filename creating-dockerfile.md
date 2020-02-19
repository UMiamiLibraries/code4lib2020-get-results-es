---
layout: page
title: Creating the Dockerfile
permalink: /creating-docker-file/
nav_order: 2
---

## Docker and Docker-Composer setup
In this workshop, you will need to have Docker and Docker-Composer installed in your computer. 
Please follow the below installation instructions from the Docker official documentation, for the operating system you are running.

<ul>
    <li>
        <a href="https://docs.docker.com/docker-for-mac/install" target="_blank">Docker for Mac</a>
    </li>
    <li>
        <a href="https://docs.docker.com/docker-for-windows/install" target="_blank">Docker for Windows 10 (Pro, Enterprise, or Education)</a>
    </li>
    <li>
        <a href="https://docs.docker.com/install/linux/docker-ce/ubuntu" target="_blank">Docker for Ubuntu</a>
    </li>
</ul>

To verify Docker is installed correctly, please run the following command on a terminal: 

{% highlight console %}
$ docker run hello-world
{% endhighlight %}

<p style="margin-top: 5px">This command should output the following:</p>

{% highlight console %}
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

{% endhighlight %}

<p style="margin-top: 5px">You should also verify docker-compose is intalled. Please run the following command in a terminal:</p>
{% highlight console %}
$ docker-compose --version
{% endhighlight %}

<p style="margin-top: 5px">Depending on the version of docker-compose you have installed, the output should look like this:</p>
{% highlight console %}
$ docker-compose version 1.25.0, build 0a186604
{% endhighlight %}

## Increase memory for Docker

For this workshop, we need to increase the amount of memory that Docker can use in the host computer. A limit of 4096 MB should be enough. 

<b>For hosts running Mac</b>

1. Open Docker settings.
2. Select the Advanced tab.
3. Increase the memory limit to 4096 MB.

<figure>
  <img src="{{ site.baseurl }}/assets/images/mac-host-settings.png" alt="Image displays Advanced settings fo Docker on Mac">
  <p class="workshop-figure-caption">Docker for Mac Advanced Settings.</p>
</figure>

<hr>

<b>For hosts running Windows</b>

1. Open Docker settings.
2. Select the Advanced tab.
3. Increase the memory limit to 4096 MB.

<figure>
  <img src="{{ site.baseurl }}/assets/images/windows-host-image.JPG" alt="Image displays Advanced settings fo Docker on Windows">
  <p class="workshop-figure-caption">Docker for Windows Advanced Settings.</p>
</figure>

<hr>

<b>For hosts running Ubuntu</b>

1. Open a terminal and run the following command:
{% highlight console %}
sudo sysctl -w vm.max_map_count=4096000
{% endhighlight %}

<hr>

Congratulations! Now you should have installed Docker and Docker-compose installed in your computer. In the next section, we will go through the steps for creating a Dockerfile that will be used to create a Docker container to host our Symfony application.

[Next: Creating the Dockerfile]({{ site.baseurl }}/docker-setup/){: .btn .btn-outline }