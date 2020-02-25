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

<p>The boilerplate structure should look like this</p>

{% highlight Docker %}
+-- ..
|-- get-results-es-boilerplate
|   |-- .docker (This directory will contain all our Docker related files)
|   |   |-- .gitkeep (This file is empty. It just helps to keep track of the empty directory)
|   |-- ingester-app (Here we will create our ingester app)
|   |   |-- composer.json (composer file with the required dependencies for the ingester app)
|   |-- search-app
|   |   |-- composer.json (composer file with the required dependencies for the search app)
|   |-- wait-for-it (We will get back to this directory in future sections of the workshop)

+-- ..
{% endhighlight %} 


<hr>

[Previous: Computer Setup]({{ site.baseurl }}/computer-setup){: .btn .btn-outline }
[Next: TBD]({{ site.baseurl }}/project-setup/){: .btn .btn-outline }