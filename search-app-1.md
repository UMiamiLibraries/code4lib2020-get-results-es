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

### A custom script to ingest the collection's data into the Elasticsearch cluster

<p>The search-app directory contains the Symfony app that will query our Elasticsearch cluster. But first, we need a way to ingest data
from the Library of Congress' collection into our cluster. Let's create it!</p>

<p>Navigate to /search-app/src/CustomScripts. This directory contains two files: ingest.php and ElasticsearchIngester.php. 
The first one will be the entry point for our script. The later is a class that will contain all the logic required for the ingestion process</p>

<p>Let's check the ElasticsearchIngester class first.</p>

<p>There is a couple of things happening in the class constructor: </p>

<p>
{% highlight php %}
# search-app/src/CustomScripts/ElasticsearchIngester.php
...
class ElasticsearchIngester
{
    private $apiUrl;
    private $elasticSearchClient;
    private $indexName;
    
    public function __construct($apiUrl, $indexName)
    {
        $this->apiUrl = $apiUrl;
        $this->indexName = $indexName;
        $elastic_host_info = [
            $_ENV['ELASTIC_HOST'],
            $_ENV['ELASTIC_PORT'],
            $_ENV['ELASTIC_USER'],
            $_ENV['ELASTIC_PASSWORD']
        ];
        $this->elasticSearchClient = ClientBuilder::create()->setHosts($elastic_host_info)->build();
    }
....
}
{% endhighlight %} 
</p>

<p>In the class constructor we are assigning values for the apiUrl and indexName variables. We are also building an 
Elasticsearch client using the official PHP client for Elasticsearch. We are reading some of the Elasticsearch settings 
from the .env file</p>

<p>
{% highlight php %}
# search-app/.env
ELASTIC_HOST=esdata-0.local
ELASTIC_PORT=9200
ELASTIC_USER=elastic
ELASTIC_PASSWORD=changeme

{% endhighlight %} 
</p>

<p>The ELASTIC_HOST value corresponds to the Elastisearch master node we defined in the docker-compose file. All the interactions
made by the PHP client for Elasticsearch will be done using port 9200. The ELASTIC_USER and ELASTIC_PASSWORD values are 
the default values. IMPORTANT: These values should be changed when using Elasticsearch in a production environment</p>

<p>There are also a couple of functions in the class that are responsible for getting the JSON data from the Library of Congress API, 
 check if a specific index in Elasticseach exists and create an index in Elasticsearch.</p>

<p>
{% highlight php %}
# search-app/src/CustomScripts/ElasticsearchIngester.php
...
private function getJsonData($url)
{
    sleep(1); //We are waiting a second between each API call to prevent being temporarily banned 
              // by the Library of Congress API
    $json = file_get_contents($url);
    return json_decode($json);
}
public function indexExists()
{
    $indexParams['index'] = $this->indexName;
    return $this->elasticSearchClient->indices()->exists($indexParams);
}
public function createIndex()
{
    $params = [
        'index' => $this->indexName
    ];
    $this->elasticSearchClient->indices()->create($params);
}
...
{% endhighlight %} 
</p>

<p>The ingestData() function deals with ingesting the data into the Elasticsearch cluster. The function goes through all
the items in the collection and gets the id, title, location, online_format, url, notes, image_url and description of each one.</p>

<p>
{% highlight php %}
# search-app/src/CustomScripts/ElasticsearchIngester.php
...
public function ingestData()
{
    $progressBar = new ProgressBar(new ConsoleOutput(), 100);
    echo 'Ingesting data' . PHP_EOL;
    $currentApiUrl = $this->apiUrl;
    do {
        $data = $this->getJsonData($currentApiUrl);
        if ($data) {
            $results = $data->results;
            $pagination = $data->pagination;
            $params = $this->prepareValues($results);
            $this->elasticSearchClient->bulk($params);
            $progressBar->advance(4);
            $currentApiUrl = !empty($pagination->next) ? $pagination->next : null;
        }
    } while (!empty($currentApiUrl));
    $progressBar->finish();
    echo PHP_EOL . 'Done ingesting data' . PHP_EOL;
}

private function prepareValues($results)
{
    $params = ['body' => []];
    foreach ($results as $result) {
        $params['body'][] = [
            'index' => [
                '_index' => $this->indexName,
                '_id' => $result->id
            ]
        ];
        $params['body'][] = [
            'title' => $result->title,
            'online_format' => $result->online_format,
            'location' => $result->location,
            'url' => $result->url,
            'notes' => $result->item->notes,
            'image_url' => $result->image_url,
            'description' => $result->description
        ];
    }
    return $params;
}
...
{% endhighlight %} 
</p>

<p>The ingest.php file creates an instance of our ElasticsearchIngester class. Then creates an index if it does not exist
and after, it ingests the collection's data into the Elasticsearch cluster</p>

<p>
{% highlight php %}
# search-app/src/CustomScripts/ingest.php
...
$apiUrl = 'https://www.loc.gov/collections/national-screening-room/?fo=json';

$elasticSearchIngester = new ElasticsearchIngester($apiUrl, 'screening-room-index');

if (!$elasticSearchIngester->indexExists()) {
    $elasticSearchIngester->createIndex();
}

$elasticSearchIngester->ingestData();
{% endhighlight %} 
</p>

<p>Now that we are familiar with the custom script to ingest the collection's data into the Elasticsearch cluster, let's run it!</p>

### Running the script to the ingest the collection's data into the Elasticsearch cluster

<p>We have to run the script from within the search-app Docker container. We are going to execute an interactive bash shell inside the container. Before
executing the following code from a console, please be sure that the containers are running.</p>

<p>
{% highlight Docker %}
docker exec -it search-app bash
{% endhighlight %} 
</p>

<p>Your console should display something similar to this: </p>

<p>
{% highlight console %}
root@19c1aaf68884:/home#
{% endhighlight %} 
</p>

<p>Every command we run now is executed from within the container. Let's switch to the CustomScripts directory and
 run the ingest.php script. Run the following two commands: </p>
<hr>

<p>
{% highlight console %}
$ cd site/wwwroot/src/CustomScripts/
$ php ingest.php
{% endhighlight %} 
</p>

<p>The output should look similar to this: </p>

<p>
{% highlight console %}
Cannot load Zend OPcache - it was already loaded
Ingesting data
 100/100 [============================] 100%
Done ingesting data
{% endhighlight %} 
</p>

<p>If you visit http://localhost:9200/_search/?pretty in your browser, you should get a response from Elasticsearch with 10 
items from the index we just created.</p>

<p>Now we have an Elasticsearch cluster with data. Time to begin query this data from our Symfony app</p>

<hr>

[Previous: The docker-compose file]({{ site.baseurl }}/docker-compose-file){: .btn .btn-outline }
[Next: The search app (part 2)]({{ site.baseurl }}/search-app-2/){: .btn .btn-outline }