# opensearch-analytics-tutorial

## A brief introduction to OpenSearch 

OpenSearch is a scalable, flexible, and extensible open-source software suite for search, analytics, and observability applications licensed under Apache 2.0. OpenSearch is used in the industry for:

- Full-text search 
- Data aggregations
- Logs, metrics and trace analysis
- Data discovery and visualisation of data

Plenty of regular databases offer search capabilities. Then why OpenSearch? 

- You need **fast** full-text search that takes into account:
  - Normalisations 
  - Synonyms
  - Stemming
  - Ranking by relevance
- You need fast data aggregations and analytics
- You want to offload the computational load from main database 
- You need fast search across big number of properties 
- You’re searching in JSON objects 
- You require auto suggestions and fuzzy search 

 Please keep in mind that as a search engine database, OpenSearch is **not** ACID compliant and only offers eventual consistency. It is also slower at deleting and modifying data when compared to a relational database.

## Goal of this tutorial

In this tutorial, our goal is to explore OpenSearch, add some data, and perform various searches on the data. OpenSearch administration including OpenSearch high-availability, security, ACLs, etc are out of scope for this tutorial.

## Prerequisites

- A self-managed or managed OpenSearch service. In this tutorial, I'm using the Aiven platform and an [Aiven for OpenSearch](https://docs.aiven.io/docs/products/opensearch) service. [Sign up for free to create an Aiven account](https://console.aiven.io/signup) and deploy an OpenSearch service to the cloud of your choice using the free credits.  
- [An Aiven authentication token](https://docs.aiven.io/docs/platform/howto/create_authentication_token)
- [Python installed](https://www.python.org/downloads/). I'm using Python 3.11.3 on macOS. 
- `opensearchpy` library installed
```shell
pip install opensearchpy
```
- An IDE of your choice. I'm using Visual Studio Code

## Create an Aiven for OpenSearch service

[Follow the instructions](https://docs.aiven.io/docs/products/opensearch/getting-started).

## Explore OpenSearch

https://docs.aiven.io/docs/products/opensearch/getting-started#access-opensearch-dashboards

Create an index.

It is not possible to send data to OpenSearch if the index doesn't exist. In OpenSearch, an index is a logical namespace that contains a collection of documents with similar characteristics. Before you can send data to OpenSearch, you need to create an index first. Sending data to a non-existent index would result in an error. Therefore, it is necessary to create the index before attempting to send data to OpenSearch.

## Add sample data

```python
import csv
from opensearchpy import OpenSearch

# OpenSearch configuration
opensearch_uri = 'YOUR OPENSEARCH URI GOES HERE'  # Replace with your OpenSearch server URI
index_name = 'YOUR OPENSEARCH INDEX NAME GOES HERE'  # Replace with the desired index name

# CSV file path
csv_file_path = 'MOCK_DATA.csv'  # Replace with the path to your CSV file

# Establish connection to OpenSearch
client = OpenSearch(opensearch_uri)

# Function to load CSV and send data to OpenSearch
def load_csv_and_send_to_opensearch(file_path):
    with open(file_path, 'r') as csv_file:
        csv_reader = csv.DictReader(csv_file)
        for row in csv_reader:
            # Send data to OpenSearch
            response = client.index(index=index_name, body=row)
            print(f"Data sent to OpenSearch: {response}")

# Load CSV and send data to OpenSearch
load_csv_and_send_to_opensearch(csv_file_path)
```

## Check if index exists

```python
import requests

def check_index_exists(uri, index_name):
    url = f"{uri}/{index_name}"
    response = requests.head(url)

    if response.status_code == 200:
        print(f"The index '{index_name}' exists.")
    elif response.status_code == 404:
        print(f"The index '{index_name}' does not exist.")
    else:
        print(f"An error occurred. Status code: {response.status_code}")

# Example usage
opensearch_uri = 'YOUR OPENSEARCH URI GOES HERE'  # Replace with your OpenSearch server URI
opensearch_index = 'YOUR OPENSEARCH INDEX NAME GOES HERE'

check_index_exists(opensearch_uri, opensearch_index)
```

## Check if a server with given name exists
