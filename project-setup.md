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

<p>We have a Git submodule in our project. It is located in the .docker/wait-for-it directory. We need to initialize the submodule and update it. First, let's run the following command in the console:</p>
<p>
{% highlight console %}
$ git submodule init
{% endhighlight %} 
</p>

<p>The previous command initialises the submodule. Next, we have to update it. Run the following code:</p>
<p>
{% highlight console %}
$ git submodule update
{% endhighlight %} 
</p>

<p>Now we have initialized the Git submodule in our project.</p>

<p>At this point, the boilerplate structure should look like this:</p>

<p>
{% highlight Docker %}
+-- ..
|-- get-results-es-boilerplate
|   |-- .docker (This directory will contain our Docker related files)
|   |   |-- wait-for-it (We will get back to this directory in future sections of the workshop)
|   |-- search-app (This directoy contains the files for our search app)
|   |-- docker-compose.yml (We will configure our application's services in this file)

+-- ..
{% endhighlight %} 
</p>

In the next section we will start creating our Dockerfile.

<hr>

[Previous: Computer Setup]({{ site.baseurl }}/computer-setup){: .btn .btn-outline }
[Next: Creating the Dockerfile]({{ site.baseurl }}/creating-dockerfile/){: .btn .btn-outline }