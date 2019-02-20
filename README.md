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

## Installation

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
sudo apt-get update && sudo apt-get install elasticsearch -y

# Edit cluster name, node name, network.host 
sudo vi /etc/elasticsearch/elasticsearch.yml

# Starting elasticsearch service
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
curl localhost:9200
```

