---
layout: page
title: Project Setup
permalink: /project-setup/
nav_order: 2
---

## Project Setup

<p>Let's get started with the project setup. We are going to clone a boilerplate project from GitHub. 
Run the following command from a console to clone the boilerplate code into your computer:</p>

<p>
{% highlight console %}
$ git clone https://github.com/UMiamiLibraries/get-results-es-boilerplate.git
{% endhighlight %} 
</p>

<p>The boilerplate structure should look like this:</p>

{% highlight Docker %}
+-- ..
|-- get-results-es-boilerplate
|   |-- .docker (This directory will contain all our Docker related files)
|   |   |-- wait-for-it (We will get back to this directory in future sections of the workshop)
|   |-- ingester-app (Here we will create our ingester app)
|   |   |-- composer.json (composer file with the required dependencies for the ingester app)
|   |-- search-app
|   |   |-- composer.json (composer file with the required dependencies for the search app)
|   |-- docker-compose.yml (We will configure our application's services in this file)

+-- ..
{% endhighlight %} 

<hr>

In the next section we will start creating our Dockerfile.


[Previous: Computer Setup]({{ site.baseurl }}/computer-setup){: .btn .btn-outline }
[Next: Creating the Dockerfile]({{ site.baseurl }}/creating-dockerfile/){: .btn .btn-outline }