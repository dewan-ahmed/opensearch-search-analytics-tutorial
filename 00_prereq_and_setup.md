## Prerequisites

- A self-managed or managed OpenSearch service. In this tutorial, I'm using the Aiven platform and an [Aiven for OpenSearch](https://docs.aiven.io/docs/products/opensearch) service. [Sign up for free to create an Aiven account](https://console.aiven.io/signup) and deploy an OpenSearch service to the cloud of your choice using the free credits.  
- [An Aiven authentication token](https://docs.aiven.io/docs/platform/howto/create_authentication_token).
- [Python installed](https://www.python.org/downloads/). I'm using Python 3.11.3 on macOS. 
- `opensearchpy` library installed.
```shell
pip install opensearchpy
```
- An IDE of your choice. I'm using Visual Studio Code.

## Create an Aiven for OpenSearch service

[Follow the instructions](https://docs.aiven.io/docs/products/opensearch/getting-started) to create an Aiven for OpenSearch service.

Once your service is up and running, [explore OpenSearch Dashboards](https://docs.aiven.io/docs/products/opensearch/getting-started#access-opensearch-dashboards).

## OpenSearch index

An OpenSearch index is a logical container within the OpenSearch search engine that holds a collection of related documents. It organizes data in a structured manner for efficient searching and retrieval. Each index is defined by a unique name and represents a specific data category or domain. Documents within an index are typically of a similar type or share common characteristics. The index configuration defines the mapping and settings for the documents it contains, such as field types, analyzers, and other indexing parameters. OpenSearch supports creating multiple indexes to separate and manage different sets of data within a search cluster.

To hold the sample data, we'll create an index called `server-logs`.

But first, let's check if index `server-logs` exists:

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
opensearch_index = 'server-logs'

check_index_exists(opensearch_uri, opensearch_index)
```