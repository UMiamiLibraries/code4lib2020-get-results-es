---
layout: page
title: The search app (part 1)
permalink: /search-app-1/
nav_order: 5
---

## The search app (part 1)

<p>
In this workshop we will be working with National Screening Room collection (https://www.loc.gov/collections/national-screening-room/) from the Library of Congress. You can find
more information about this collection 
<a href="https://www.loc.gov/collections/national-screening-room/about-this-collection/" target="_blank">here</a>.
</p>

<p>We will ingest data from this collection into our Elasticsearch cluster. Then, we will use our application to find items 
in our cluster from this collection.</p>

<p>The API from the Library of Congress is still a work in progress. The Beta documentation can be found 
<a href="https://libraryofcongress.github.io/data-exploration/" target="_blank">here</a>. According to the documentation, 
we can the collection in JSON format by appending ?fo=json to the URL of the collection. </p>

<p>If you open <a href="https://www.loc.gov/collections/national-screening-room/?fo=json" 
target="_blank">https://www.loc.gov/collections/national-screening-room/?fo=json</a> in your browser you will get the
 JSON of this collection.</p>

### A custom script to ingest the collection into our Elasticsearch cluster

<hr>

[Previous: The docker-compose file]({{ site.baseurl }}/docker-compose-file){: .btn .btn-outline }
[Next: TBD]({{ site.baseurl }}/creating-dockerfile/){: .btn .btn-outline }