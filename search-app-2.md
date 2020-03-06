---
layout: page
title: The search app (part 2)
permalink: /search-app-2/
nav_order: 6
---

## The search app (part 2)

<p>Let's start by defining the SearchController in our Symfony app</p>

<p>The SearchController class currently have a defined route for the root URL of our application. This route calls the 
index() function which returns a twig template. We get this when visiting the following URL http://localhost:8080/public/</p>

<p>
<figure>
  <img src="{{ site.baseurl }}/assets/images/search-app-running-from-docker-container.png" alt="Image displays Search app running on the Docker Container">
</figure>
</p>

<p>Let's modify the controller class so we can do a search. Replace the index() function with the following one. 
Be sure to import all the required classes.</p>

<p>
{% highlight php %}
# search-app/src/Controller/SearchController.php
    /**
     * @Route("/")
     */
    public function index(Request $request)
    {
        $searchTerm = $request->query->get('query');
        $searchResults = [];

        if ($searchTerm) {
            $queryResponse = $this->doSearch($searchTerm);
            $searchResults = $queryResponse['hits']['hits'];
        }

        return $this->render('search-app.html.twig', [
            'searchTerm' => $searchTerm,
            'searchResults' => $searchResults
        ]);
    }
{% endhighlight %} 
</p>

<p>The index() function will render a twig template with the results from our query. We need to define the doSearch() auxiliary function that is being called from the index() function. 
Add the following code below the index() function: </p>

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

        $searchParams['index'] = 'screening-room-index'; // which index to search
        $searchParams['body']['query']['simple_query_string']['query'] = $searchTerm; // what to search for

        return $elasticSearchClient->search($searchParams);
    }
{% endhighlight %} 
</p>

<p>In the doSearch() function, we initialize an Elasticsearch client (using the values stored in the .env file) and then 
we prepare a simple query string query and return the results.</p>

<p>At this point, your SearchController class should look like this:</p>
<p>
{% highlight PHP %}
# search-app/src/Controller/SearchController.php
namespace App\Controller;

use Elasticsearch\ClientBuilder;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

class SearchController extends AbstractController
{
    /**
     * @Route("/")
     */
    public function index(Request $request)
    {
        $searchTerm = $request->query->get('query');
        $searchResults = [];

        if ($searchTerm) {
            $queryResponse = $this->doSearch($searchTerm);
            $searchResults = $queryResponse['hits']['hits'];
        }

        return $this->render('search-app.html.twig', [
            'searchTerm' => $searchTerm,
            'searchResults' => $searchResults
        ]);
    }

    private function doSearch($searchTerm)
    {
        $elastic_host_info = [
            $_ENV['ELASTIC_HOST'],
            $_ENV['ELASTIC_PORT']
        ];

        $elasticSearchClient = ClientBuilder::create()->setHosts($elastic_host_info)->build();

        $searchParams['index'] = 'screening-room-index'; // which index to search
        $searchParams['body']['query']['simple_query_string']['query'] = $searchTerm; // what to search for

        return $elasticSearchClient->search($searchParams);
    }
}
{% endhighlight %} 
</p>

<p>You can find the search-app twig template in search-app/templates/search-app.html.twig.</p>

### Let's search!

<p>Now we should be ready to test our app. Open this URL in your browser http://localhost:8080/public/. You should see 
a search bar. Type a search term and hit Enter. Depending on the search term, you should get results like the ones below: </p>

<p>
<figure>
  <img src="{{ site.baseurl }}/assets/images/search-results.png" alt="Image displays an example of a search result">
</figure>
</p>


<hr>

[Previous: The search app (part 1)]({{ site.baseurl }}/search-app-1/){: .btn .btn-outline }
[Next: Searching multiple indexes]({{ site.baseurl }}/searching-multiple-indexes/){: .btn .btn-outline }