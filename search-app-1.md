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

### A custom command to ingest the collection's data into the Elasticsearch cluster

<p>The search-app directory contains the Symfony app that will query our Elasticsearch cluster. But first, we need a way to ingest data
from the Library of Congress' collection into our cluster.</p>

<p>Navigate to /search-app/src/Command. This directory contains a class named ElasticsearchIngesterCommand. 
The class defines custom Symfony command that will ingest data into our Elasticsearch cluster. Let's give it a look.</p>

<p>Our class extends the Command class of Symfony and overrides the configure() and execute() functions.</p>

<p>
{% highlight php %}
# search-app/src/CustomScripts/ElasticsearchIngesterCommand.php
...
class ElasticsearchIngesterCommand extends Command
{
    protected static $defaultName = 'app:ingest-data';
    private $elasticSearchClient;
    private $outputInterface;
    private $apiUrl;
    private $indexName;
    
    public function __construct()
    {
        $elastic_host_info = [
            $_ENV['ELASTIC_HOST'],
            $_ENV['ELASTIC_PORT'],
            $_ENV['ELASTIC_USER'],
            $_ENV['ELASTIC_PASSWORD']
        ];
        $this->elasticSearchClient = ClientBuilder::create()->setHosts($elastic_host_info)->build();
        parent::__construct();
    }
        
    protected function configure()
    {
        $this->setDescription('Ingests data from a LOC Collection API into an Elasticsearch cluster')
            ->addArgument('apiUrl', InputArgument::REQUIRED, 'Pass API Url')
            ->addArgument('indexName', InputArgument::REQUIRED, 'Pass the index name');
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $this->outputInterface = $output;
        $this->apiUrl = $input->getArgument('apiUrl');
        $this->indexName = $input->getArgument('indexName');

        $output->writeln('Checking if index exists...');
        if (!$this->indexExists()) {
            $output->writeln('Creating index ' . $this->indexName);
            $this->createIndex();
        }

        $this->ingestData();
        return 0;
    }
....
}
{% endhighlight %} 
</p>

<p>In the class constructor we are building an Elasticsearch client using the official PHP client for Elasticsearch. 
The Elasticsearch settings are stored in the .env file</p>

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
private function ingestData()
{
    $this->outputInterface->writeln('Ingesting data');
    $progressBar = new ProgressBar($this->outputInterface, 100);
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
    $this->outputInterface->writeln( PHP_EOL . 'Finished ingesting data');
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
            'title' => isset($result->title) ? $result->title : '',
            'online_format' => isset($result->online_format) ? $result->online_format : '',
            'location' => isset($result->location) ? $result->location : '',
            'url' => isset($result->url) ? $result->url : '',
            'notes' => isset($result->item->notes) ? $result->item->notes : '',
            'image_url' => isset($result->image_url) ? $result->image_url : '',
            'description' => isset($result->description) ? $result->description : ''
        ];
    }
    return $params;
}
...
{% endhighlight %} 
</p>

<p>Now that we are familiar with the custom command to ingest the collection's data into the Elasticsearch cluster, let's run it!</p>

### Running the command to ingest the collection's data into the Elasticsearch cluster

<p>We have to run the command from within the search-app Docker container. We can do this by executing an interactive 
bash shell inside the container. Before executing the following code from a console, please be sure that the containers are running.</p>

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

<p>Every command we run now is executed from within the container. Let's switch to the root directory of the Symfony app and
 run the custom command. Run the following two commands: </p>
<hr>

<p>
{% highlight console %}
$ cd site/wwwroot/
$ php bin/console app:ingest-data 'https://www.loc.gov/collections/national-screening-room/?fo=json' 'screening-room-index'
{% endhighlight %} 
</p>

<p>The output should look similar to this: </p>

<p>
{% highlight console %}
Cannot load Zend OPcache - it was already loaded
Checking if index exists...
Ingesting data
 100/100 [============================] 100%
Finished ingesting data
{% endhighlight %} 
</p>

<p>If you visit http://localhost:9200/_search/?pretty in your browser, you should get a response from Elasticsearch with 10 
items from the index we just created.</p>

<p>Now we have an Elasticsearch cluster with data. Time to begin querying this data from our Symfony app</p>

<hr>

[Previous: The docker-compose file]({{ site.baseurl }}/docker-compose-file){: .btn .btn-outline }
[Next: The search app (part 2)]({{ site.baseurl }}/search-app-2/){: .btn .btn-outline }