---
layout: page
title: Searching multiple indexes
permalink: /searching-multiple-indexes/
nav_order: 7
---

## Searching multiple indexes

<p>We can search multiple indexes in Elasticsearch. Let's ingest the 
<a href="https://www.loc.gov/collections/railroad-maps-1828-to-1900" target="_blank">Railroad Maps, 1828 to 1900
</a> collection from the Library of Congress into the Elasticsearch cluster. Run the following command from the console:</p>

<p>
{% highlight console %}
$ cd site/wwwroot/
$ php bin/console app:ingest-data 'https://www.loc.gov/collections/railroad-maps-1828-to-1900/?fo=json' 'railroad-maps-index'
{% endhighlight %} 
</p>

<p>Let's verify that the new index was added by opening http://localhost:9200/_cat/indices?v in your browser. 
You should get something similar to this:</p>

<p>
{% highlight JSON %}
health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   screening-room-index     Aia1GChwQKOizP1VOLV46w   1   1        366            0      1.5mb          798kb
green  open   railroad-maps-index      kaGevN-qTNSRA6fWCSVeag   1   1        635            0      2.1mb            1mb
{% endhighlight %} 
</p>

<p>We need to tell our app to search in both indexes. We do this by adding the new index to the searchParams array of the 
doSearch() function in the SearchController class. The function should look like this:</p>

<p>
{% highlight php %}
# search-app/src/Controller/SearchController.php

    private function doSearch($searchTerm)
    {
        $elastic_host_info = [
            $_ENV['ELASTIC_HOST'],
            $_ENV['ELASTIC_PORT']
        ];

        $elasticSearchClient = ClientBuilder::create()->setHosts($elastic_host_info)->build();

        $searchParams['index'] = 'screening-room-index,railroad-maps-index'; // which index to search
        $searchParams['body']['query']['simple_query_string']['query'] = $searchTerm; // what to search for

        return $elasticSearchClient->search($searchParams);
    }
{% endhighlight %} 
</p>

<Let's also output in our search results the name of the index the result belongs to. Add the following code after line 39 
in the search-app.html.twig template

<p>
{% highlight php %}
# search-app/src/Controller/SearchController.php
<p>Index name: {% raw %}{{ result._index }}</p>{% endraw %}
{% endhighlight %} 
</p>

<p>Now if you do a search, you should get results from both indexes.</p>
<hr>

[Previous: The search app (part 2)]({{ site.baseurl }}/search-app-2/){: .btn .btn-outline }
[Next: The Dev Tools Console]({{ site.baseurl }}/dev-tools-console/){: .btn .btn-outline }