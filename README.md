# Elasticsearch - POC
Setting up ELK stack cluster with Vagrant

## Prerequisites

* Virtualbox
* Vagrant
* JAVA 8
* Internet Connection

## What is Elasticsearch?

Elasticsearch is a highly scalable open-source full-text search and analytics engine. It allows you to store, search, and analyze big volumes of data quickly and in near real time.

## Basic Concepts

* Near Real Time (NRT) - Elasticsearch is a near-realtime search platform. What this means is there is a slight latency (normally one second) from the time you index a document until the time it becomes searchable.

* Cluster - A cluster is a collection of one or more nodes (servers) that together holds your entire data and provides federated indexing and search capabilities across all nodes

* Node - A node is a single server that is part of your cluster, stores your data, and participates in the cluster’s indexing and search capabilities.

* Index - An index is a collection of documents that have somewhat similar characteristics. For example, you can have an index for customer data, another index for a product catalog, and yet another index for order data.

* Document - A document is a basic unit of information that can be indexed. For example, you can have a document for a single customer, another document for a single product, and yet another for a single order.

* Shards & Replicas - Elasticsearch provides the ability to subdivide your index into multiple pieces called shards.Elasticsearch allows you to make one or more copies of your index’s shards into what are called replica shards, or replicas for short

## Installation of Elasticsearch and Kibana

To setup ELK cluster we need to instal JAVA, add ELK repo and install it. Run the below commands to install ELK stack

```bash
# Login into vagrant machine
vagrant up
vagrant ssh
sudo apt update
sudo apt upgrade -y

# Install JAVA
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer -y
java -version
update-alternatives --config java
sudo vi /etc/environment
JAVA_HOME="/usr/lib/jvm/java-8-oracle"
source /etc/environment
echo $JAVA_HOME

# Install Elasticsearch
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https -y
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt-get update && sudo apt-get install elasticsearch kibana -y

# Edit cluster name, node name, network.host 
sudo vi /etc/elasticsearch/elasticsearch.yml

# Edit server.host: 0.0.0.0
sudo vi /etc/kibana/kibana.yml

# Starting elasticsearch service
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch

# Starting kibana
sudo systemctl enable kibana
sudo systemctl start kibana

# Test Elasticsearch and kibana
curl localhost:9200
curl localhost:5601/app/kibana

# Check Cluster health
curl -H "Content-Type: application/json" -XGET localhost:9200/_cat/health?v
```

## Exploring Elasticsearch cluster with CRUD operation

* Go to kibana dashboard and click "Dev Tools". It gives you the option to run CURL commands

```bash
GET _cat/health?v
GET /_cat/nodes?v
GET /_cat/indices?v
PUT /customer?pretty
GET /_cat/indices?v
# customer index will be in yellow as it has only one node[No high availabilty]
GET /_cat/indices?v&index=customer

# write document on customer index
PUT customer/_doc/1?pretty
{
  "name": "John Doe"
}
GET /customer/_doc/1?pretty

# Delete Index
DELETE /customer?pretty
GET /_cat/indices?v&index=customer # Not found error

# Update data
PUT customer/_doc/1?pretty
{
  "name": "Akilan"
}
GET /customer/_doc/1?pretty

# Post data without id
POST /customer/_doc?pretty
{
  "name": "Jane Doe"
}
# Update using POST method
POST /customer/_doc/1/_update?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}

# Update by script
POST customer/_doc/1/_update?pretty
{
  "script": "ctx._source.age += 5"
}
# Elasticsearch provides the ability to update multiple documents given a query condition (like an SQL UPDATE-WHERE statement). See docs-update-by-query API

# Delete document
DELETE /customer/_doc/1?pretty

# Bulk insert
POST /customer/_doc/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }

# Bulk Update and delete
POST /customer/_doc/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}

# https://www.json-generator.com/ - place to generate some random JSON data
# Insert data from Json file
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
```
## Exploring Search API

* There are two basic ways to run searches: one is by sending search parameters through the REST request URI and the other by sending them through the REST request body
  
```bash
# search by REST API call
GET bank/_search?q=*&sort=account_number:asc&pretty

# Search by REST request body

GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "account_number": {
        "order": "asc"
      }
    }
  ],
  "from": 0, 
  "size": 10
}

# return two fields, account_number and balance
GET bank/_search
{
  "query": {
    "match": {
      "account_number": "20"
    }
  },
  "_source": ["account_number","balance"]
}

# Below query returns document if address has "mill" or "lane"

GET bank/_search
{
  "query": {
    "match": {
      "address": "mill lane"
    }
  }
}

# Below query returns document if address has "mill lane" phrase
GET bank/_search
{
  "query": {
    "match_phrase": {
      "address": "mill lane"
    }
  }
}

# Boolean query allows us to compose smaller queries into bigger queries using boolean logic
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match" : { "age" : 32 }
        }
      ],
      "must_not": [
        {
          "match": {
            "gender": "M"
          }
        }
      ]
    }
  }
}

# Filter Female between age [23-30] with balance 20k-30k
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        { 
          "match": {"gender": "F"}
        },
        {
          "range": {
            "age": {
              "gte": 23,
              "lte": 30
             }
          }
        },
        {
          "range": {
            "balance": {
              "gte": 20000,
              "lte": 30000
           }
          }
        }
      ]
    }
    
  }
}

# Aggregation provide the ability to group and extract statistics from your data
# State wise number of accounts
GET bank/_search
{
  "size":0, 
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
# Average balance 
GET bank/_search
{
  "size":0, 
  "aggs": {
    "avg_balance": {
      "avg": {
        "field": "balance"
      }
    }
  }
}
```