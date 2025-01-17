<p align="center"><img src="https://github.com/elastic/site-search-ruby/blob/master/logo-site-search.png?raw=true" alt="Elastic Site Search Logo"></p>

<p align="center"><a href="https://circleci.com/gh/elastic/site-search-ruby"><img src="https://circleci.com/gh/elastic/site-search-ruby.svg?style=svg" alt="CircleCI build"></a></p>

> A first-party Ruby client for the [Elastic Site Search API](https://swiftype.com/documentation/site-search/overview).

## Contents

- [Getting started](#getting-started-)
- [Usage](#usage)
- [Migrating from pervious versions](#migrating-from-previous-versions)
- [Development](#development)
- [FAQ](#faq-)
- [Contribute](#contribute-)
- [License](#license-)

---

## Getting started 🐣

Before beginning with the `elastic-site-search` gem, you should be familiar with the concepts behind the Elastic Site Search API:

1. **Engines**
2. **DocumentTypes**
3. **Documents**

An **Engine** is a search engine.

It can contain one or more **DocumentTypes** which are collections of **Documents**.

A **Document** is a collection of fields that can be queried using the Site Search API.

Documents have a special **external_id** field that ties a Document in Site Search to a record in your system. The layout of fields of the Documents belonging to a DocumentType is called a **schema**. Fields may be strings, integers, geographic locations, and so forth.

The Documents in your Engine can be searched two ways: **full-text** (`search`) or **autocomplete** (`suggest`). The difference is that autocomplete queries work on prefixes (for example, "gla" will match "glass"). This is less accurate in general, but is useful for implementing type-ahead search drop downs.

You can think of an Engine as a database, DocumentTypes as tables, and Documents as rows. Using the API, you can search an Engine for all Documents containing a word. You can also search an individual DocumentType, or any subset of DocumentTypes.

The examples in this documentation use the schema defined in the [site-search-api-example](https://github.com/swiftype/swiftype-api-example) project, which is based on YouTube. It has two DocumentTypes, **videos** and **channels**. Using the script found in the site-search-api-example project, you can create your own search engine that matches the examples and try the queries for yourself.

To learn more about the Site Search API, read the [API overview](https://swiftype.com/documentation/site-search/overview) and our [schema design tutorial](https://swiftype.com/documentation/site-search/guides/schema-design).

Depends on Ruby.

To install the gem, execute:

    gem install elastic-site-search

Or place `gem 'elastic-site-search', '~> 2.0.0` in your `Gemfile` and run `bundle install`.

> **Note:** This client has been developed for the [Elastic Site Search](https://www.elastic.co/products/site-search/service) API endpoints only.

## Usage

### Configuration:

Before issuing commands to the API, configure the client with your API key:

    Elastic::SiteSearch.api_key = 'YOUR_API_KEY'

You can find your API key in your [Account Settings](https://app.swiftype.com/settings/account).

### Create a client

    client = Elastic::SiteSearch::Client.new

You can also provide the API key when creating the client instance:

    client = Elastic::SiteSearch::Client.new(:api_key => 'different_api_key')

If the API key is provided as an option to constructor, it will override the globally configured Elastic::SiteSearch API key (if any).

### Specifying an HTTP Proxy

    client = Elastic::SiteSearch::Client.new(:api_key => 'api_key', :proxy => 'http://localhost:8888')

This client will also support configuring a proxy via the environment variable `http_proxy`.

### Full-text search

If you want to search for `cat` on your engine, you can use:

    results = client.search('site-search-api-example', 'cat')
    results['videos']   # => [{'external_id' => 'QH2-TGUlwu4', 'title' => 'Nyan Cat [original]', ... }, ... ]
    results['channels'] # => [{'external_id' => 'UC3VHfy8e1jbDnT5TG2pjP1w', 'title' => 'saraj00n', ... }, ... ]

To limit the search to only the `videos` DocumentType:

    results = client.search_document_type('site-search-api-example', 'videos', 'cat')
    results['videos']   # => [{'external_id' => 'QH2-TGUlwu4', 'title' => 'Nyan Cat [original]', ... }, ... ]
    results['channels'] # => nil

Both search methods allow you to specify options to filter or sort on fields, boost the weight of certain fields, calculate faceted counts, and so on. For more details on these options, review the [Search Options](https://swiftype.com/documentation/site-search/searching).

Here is an example for showing only videos in the "Pets & Animals" category (which has ID 23):

    results = client.search_document_type('site-search-api-example', 'videos', 'cat', {:filters => {'videos' => {:category_id => '23'}}})

### Autocomplete search

Autocomplete (also known as suggest, prefix, or type-ahead) searches can be used to implement autocompletion. They have the same functionality as full-text searches, but work on prefixes instead of all the text. Because of this, autocomplete queries only search string fields.

You can perform a suggest query across all of your Engine's Documents:

    results = client.suggest("site-search-api-example", "gla")
    results['videos'] # => [{'external_id' => 'v1uyQZNg2vE', 'title' => 'How It Feels [through Glass]', ...}, ...]

or just for one DocumentType:

    results = client.suggest_document_type("site-search-api-example", "videos", "gla")
    results['videos'] # => [{'external_id' => 'v1uyQZNg2vE', 'title' => 'How It Feels [through Glass]', ...}, ...]

or add options to have more control over the results:

    results = client.suggest('site-search-api-example', 'glass', {:sort_field => {'videos' => 'view_count'}, :sort_direction => {'videos' => 'desc'}})

### Engines

Retrieve every Engine:

    engines = client.engines

Create a new Engine with the name `site-search-api-example`:

    engine = client.create_engine('site-search-api-example')

Retrieve an Engine by `slug` or `id`:

    engine = client.engine('site-search-api-example')
    engine = client.engine('5230b9102ed960ba20000021')

Delete an Engine by `slug` or the `id`:

    client.destroy_engine('site-search-api-example')

### Document Types

List all the

Retrieve `DocumentTypes`s of the Engine with the `slug` field `site-search-api-example`:

    document_types = client.document_types('site-search-api-example')

Show the second batch of documents:

    document_types = client.document_types('site-search-api-example', 2)

Create a new DocumentType for an Engine with the name `videos`:

    document_type = client.create_document_type('site-search-api-example', 'videos')

Retrieve an DocumentType by `slug` or `id`:

    document_type = client.document_type('site-search-api-example', 'videos')

Delete a DocumentType using the `slug` or `id` of it:

    client.destroy_document_type('site-search-api-example', 'videos')

### Documents

Retrieve the first page of Documents of Engine `site-search-api-example` and DocumentType `videos`:

    documents = client.documents('site-search-api-example', 'videos')

Retrieve a specific Document using its `id` or `external_id`:

    document = client.document('site-search-api-example', 'videos', 'FHtvDA0W34I')

To create or update. single or multiple documents, we use the method `index_documents`.

Create a new Document with mandatory `external_id` and user-defined fields:

    document = client.index_documents('site-search-api-example', 'videos', {
        :external_id => 'FHtvDA0W34I',
        :fields => [
            {:name => 'title', :value => "Felix Baumgartner's supersonic freefall from 128k' - Mission Highlights", :type => 'string'},
            {:name => 'url', :value => 'http://www.youtube.com/watch?v=FHtvDA0W34I', :type => 'enum'},
            {:name => 'chanel_id', :value => 'UCblfuW_4rakIf2h6aqANefA', :type => 'enum'}
        ]})

Create multiple Documents at once and return status for each Document creation:

    response = client.index_documents('site-search-api-example', 'videos', [{
        :external_id => 'FHtvDA0W34I',
        :fields => [
            {:name => 'title', :value => "Felix Baumgartner's supersonic freefall from 128k' - Mission Highlights", :type => 'string'},
            {:name => 'url', :value => 'http://www.youtube.com/watch?v=FHtvDA0W34I', :type => 'enum'},
            {:name => 'chanel_id', :value => 'UCblfuW_4rakIf2h6aqANefA', :type => 'enum'}
        ]}, {
        :external_id => 'dMH0bHeiRNg',
        :fields => [
            {:name => 'title', :value => 'Evolution of Dance - By Judson Laipply', :type => 'string'},
            {:name => 'url', :value => 'http://www.youtube.com/watch?v='dMH0bHeiRNg', :type => 'enum'},
            {:name => 'chanel_id', :value => UC5B9H4l2vtgo7cAoExcFh-w', :type => 'enum'}
        ]}])

Update fields of an existing Document specified by `id` or `external_id`:

    client.index_documents('site-search-api-example', 'videos', 'FHtvDA0W34I', {:title =>'New Title'})

**NOTE:** A field must already exist on a Document in order to update it.

Update multiple Documents at once:

    response = client.index_documents('site-search-api-example','videos', [
        {:external_id => 'FHtvDA0W34I', :fields => {:view_count => 32874417}},
        {:external_id => 'dMH0bHeiRNg', :fields => {:view_count => 98323493}}
    ])

All methods above will have a return in the following format:

    [
      {
        "id": "5473d6142ed96065a9000001",
        "external_id": "FHtvDA0W34I",
        "status": "complete",
        "errors": [],
        "links": {
          "receipt": "https://api.swiftype.com/api/v1/document_receipts/5473d6142ed96065a9000001.json",
          "document": "https://api.swiftype.com/api/v1/engine/xyz/document_type/abc/document/5473d6142ed96065a9000001.json"
        }
      },
      {
        "id": "5473d6142ed96065a9000002",
        "external_id": "dMH0bHeiRNg",
        "status": "complete",
        "errors": [],
        "links": {
          "receipt": "https://api.swiftype.com/api/v1/document_receipts/5473d6142ed96065a9000001.json",
          "document": "https://api.swiftype.com/api/v1/engine/xyz/document_type/abc/document/5473d6142ed96065a9000002.json"
        }
      }
    ]

**NOTE:** If you'd like to create or update documents asynchronously, simply pass the option `:async => true` as the last argument.

For instance, to Create multiple Documents at once:

    response = client.index_documents('site-search-api-example', 'videos', [{
        :external_id => 'FHtvDA0W34I',
        :fields => [
            {:name => 'title', :value => "Felix Baumgartner's supersonic freefall from 128k' - Mission Highlights", :type => 'string'},
            {:name => 'url', :value => 'http://www.youtube.com/watch?v=FHtvDA0W34I', :type => 'enum'},
            {:name => 'chanel_id', :value => 'UCblfuW_4rakIf2h6aqANefA', :type => 'enum'}
        ]}, {
        :external_id => 'dMH0bHeiRNg',
        :fields => [
            {:name => 'title', :value => 'Evolution of Dance - By Judson Laipply', :type => 'string'},
            {:name => 'url', :value => 'http://www.youtube.com/watch?v='dMH0bHeiRNg', :type => 'enum'},
            {:name => 'chanel_id', :value => UC5B9H4l2vtgo7cAoExcFh-w', :type => 'enum'}
        ]}],
        :async => true )
        #=> {
          "batch_link": "https://api.swiftype.com/api/v1/document_receipts.json?ids=5473d6142ed96065a9000001,5473d6142ed96065a9000002",
          "document_receipts": [
            {
              "id": "5473d6142ed96065a9000001",
              "external_id": "FHtvDA0W34I",
              "status": "pending",
              "errors": [],
              "links": {
                "receipt": "https://api.swiftype.com/api/v1/document_receipts/5473d6142ed96065a9000001.json",
                "document": null
              }
            },
            {
              "id": "5473d6342ed96065a9000002",
              "external_id": "dMH0bHeiRNg",
              "status": "pending",
              "errors": [],
              "links": {
                "receipt": "https://api.swiftype.com/api/v1/document_receipts/5473d6142ed96065a9000002.json",
                "document": null
              }
            }
          ]
        }

To check the status of documents with their document_receipt ids:

    response = client.document_receipts(["5473d6142ed96065a9000001", "5473d6342ed96065a9000002"])
    #=> [
      {
        "id": "5473d6142ed96065a9000001",
        "external_id": "FHtvDA0W34I",
        "status": "complete",
        "errors": [],
        "links": {
          "receipt": "https://api.swiftype.com/api/v1/document_receipts/5473d6142ed96065a9000001.json",
          "document": "https://api.swiftype.com/api/v1/engine/xyz/document_type/abc/document/5473d6142ed96065a9000001.json"
        }
      },
      {
        "id": "5473d6142ed96065a9000002",
        "external_id": "dMH0bHeiRNg",
        "status": "complete",
        "errors": [],
        "links": {
          "receipt": "https://api.swiftype.com/api/v1/document_receipts/5473d6142ed96065a9000001.json",
          "document": "https://api.swiftype.com/api/v1/engine/xyz/document_type/abc/document/5473d6142ed96065a9000002.json"
        }
      }
    ]

Destroy a Document by external_id:

    client.destroy_document('site-search-api-example','videos','dFs9WO2B8uI')

Destroy multiple Documents at once:

    response = client.destroy_documents('site-search-api-example', 'videos', ['QH2-TGUlwu4, 'v1uyQZNg2vE', 'ik5sdwYZ01Q'])
    #=> [true, true, true]

### Domains

**Domains** are a feature of crawler-based engines that represent a web site to index with the Swiftype web crawler.

Retrieve all Domains of Engine `websites`:

    domains = client.domains('websites')
    #=> [{"id"=>"513fcc042ed960c186000001", "engine_id"=>"513fcc042ed960114400000d", "submitted_url"=>"http://example.com/", "start_crawl_url"=>"http://www.example.com/", "crawling"=>false, "document_count"=>337, "updated_at"=>"2013-03-13T00:48:32Z"}, ...]

Retrieve a specific Domain by `id`:

    domain = client.domain('websites', '513fcc042ed960c186000001'')
    #=> {"id"=>"513fcc042ed960c186000001", "engine_id"=>"513fcc042ed960114400000d", "submitted_url"=>"http://example.com/", "start_crawl_url"=>"http://www.example.com/", "crawling"=>false, "document_count"=>337, "updated_at"=>"2013-03-13T00:48:32Z"}

Create a new Domain with the URL `https://swiftype.com` and start crawling:

    domain = client.create_domain('websites', 'https://swiftype.com')

Delete a Domain using its `id`:

    client.destroy_domain('websites', '513fcc042ed960c186000001')

Initiate a recrawl of a specific Domain using its `id`:

    client.recrawl_domain('websites', '513fcc042ed960c186000001')

Add or update a URL for a Domain:

    client.crawl_url('websites', '513fcc042ed960c186000001', 'https://swiftype.com/new/path.html')

### Analytics

Site Search records the number of searches, autoselects (when a user clicks a link from a suggest result), and clickthroughs (when a user clicks through to an item from a search result list. You can view this information in your Site Search Dashboard, but you can also export it using the API.

To get the number of searches per day from an Engine in the last 14 days:

    searches = client.analytics_searches('site-search-api-example')
    #=> [['2013-09-13', '123'], [2013-09-12', '94'], ... ]

You can also use a specific start and/or end date:

    searches = client.analytics_searches('site-search-api-example', {:start_date => '2013-01-01', :end_date => '2013-01-07'})

To get the number of autoselects in the past 14 days:

    autoselects = client.analytics_autoselects('site-search-api-example')

As with searches you can also limit by start and/or end date:

    autoselects = client.analytics_autoselects('site-search-api-example', {:start_date => '2013-01-01', :end_date => '2013-01-07'})

If you are interested in the top queries for your Engine you can use:

    top_queries = client.analytics_top_queries('site-search-api-example')
    # => [['query term', 123], ['another query', 121], ['yet another query', 92], ...]

To see more top queries you can paginate through them using:

    top_queries = client.analytics_top_queries('site-search-api-example', {:page => 2})

Or you can get the top queries in a specific date range:

    top_queries = client.analytics_top_queries('site-search-api-example', {:start_date => '2013-01-01', :end_date => '2013-01-07'})

If you want to improve you search results, you should always have a look at search queries, that return no results and perhaps add some Documents that match for this query or use our pining feature to add Documents for this query:

    top_no_result_queries = client.analytics_top_no_result_queries('site-search-api-example')

You can also specifiy a date range for queries without results:

    top_no_result_queries = client.analytics_top_no_result_queries('site-search-api-example', {:start_date => '2013-01-01', :end_date => '2013-01-07'})

## Development

You can run tests with `rspec`.

All HTTP interactions are stubbed out using VCR.

## FAQ 🔮

### Where do I report issues with the client?

If something is not working as expected, please open an [issue](https://github.com/elastic/site-search-ruby/issues/new).

### Where can I learn more about Site Search?

Your best bet is to read the [documentation](https://swiftype.com/documentation/site-search).

### Where else can I go to get help?

You can checkout the [Elastic Site Search community discuss forums](https://discuss.elastic.co/c/site-search).

## Contribute 🚀

We welcome contributors to the project. Before you begin, a couple notes...

- Before opening a pull request, please create an issue to [discuss the scope of your proposal](https://github.com/elastic/site-search-ruby/issues).
- Please write simple code and concise documentation, when appropriate.

## License 📗

[Apache 2.0](https://github.com/elastic/site-search-ruby/blob/master/LICENSE.txt) © [Elastic](https://github.com/elastic)

Thank you to all the [contributors](https://github.com/elastic/site-search-ruby/graphs/contributors)!
