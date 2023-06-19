# opensearch-analytics-tutorial

# A brief introduction to OpenSearch 

[OpenSearch](https://opensearch.org/) is a scalable, flexible, and extensible open-source software suite for search, analytics, and observability applications licensed under Apache 2.0. OpenSearch is used in the industry for:

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

# Goal and prerequisites

In this tutorial, our goal is to explore OpenSearch, add some data, and perform various searches and analysis on the data. OpenSearch administration including OpenSearch high-availability, security, ACLs, etc are out of scope for this tutorial.

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

## Generate sample data

The following Python code generates 1000 fake records using Python faker libary and adds the data to a JSON file. A sample record looks like this:

```json
{
        "server_id": "8297def4-7fca-41f4-9368-e48c3c6b24c3",
        "server_ip": "66.125.35.51",
        "server_mac": "c9:87:2a:b0:ab:f3",
        "server_location": "Toddburgh",
        "server_memory_used": 78,
        "server_temperature": 63,
        "server_os": "Fedora",
        "server_start_time": "03/18/2023 08:58:12",
        "hardware": {
            "type": "Memory",
            "capacity": 1,
            "vendor": "Nelson-Morton"
        },
        "security": {
            "firewall_rules": [
                {
                    "port": 5802,
                    "protocol": "UDP",
                    "access": "DENY"
                },
                {
                    "port": 53825,
                    "protocol": "TCP",
                    "access": "ALLOW"
                },
                {
                    "port": 2810,
                    "protocol": "TCP",
                    "access": "DENY"
                },
                {
                    "port": 9045,
                    "protocol": "UDP",
                    "access": "DENY"
                },
                {
                    "port": 62976,
                    "protocol": "UDP",
                    "access": "ALLOW"
                }
            ],
            "security_events": [
                {
                    "type": "Login Attempt",
                    "date": "06/05/2023 12:41:32"
                },
                {
                    "type": "Intrusion Detection",
                    "date": "06/10/2023 21:27:32"
                },
                {
                    "type": "Authentication Failure",
                    "date": "06/18/2023 03:41:03"
                },
                {
                    "type": "Login Attempt",
                    "date": "06/11/2023 02:52:30"
                },
                {
                    "type": "Authentication Failure",
                    "date": "06/05/2023 10:37:16"
                },
                {
                    "type": "Intrusion Detection",
                    "date": "06/10/2023 18:21:06"
                }
            ]
        }
    }
```

Let's generate the sample data:

```python
import json
import random
from faker import Faker

fake = Faker()

# Generate nested hardware data
def generate_hardware_data():
    hardware_type = random.choice(['CPU', 'Memory', 'Storage'])
    hardware_capacity = random.randint(1, 10)
    hardware_vendor = fake.company()
    hardware_data = {
        "type": hardware_type,
        "capacity": hardware_capacity,
        "vendor": hardware_vendor
    }
    return hardware_data

# Generate nested security data
def generate_security_data():
    firewall_rules = []
    for _ in range(random.randint(1, 5)):
        port = random.randint(1, 65535)
        protocol = random.choice(['TCP', 'UDP'])
        access = random.choice(['ALLOW', 'DENY'])
        firewall_rule = {
            "port": port,
            "protocol": protocol,
            "access": access
        }
        firewall_rules.append(firewall_rule)

    security_events = []
    for _ in range(random.randint(1, 10)):
        event_type = random.choice(['Login Attempt', 'Authentication Failure', 'Intrusion Detection'])
        event_date = fake.date_time_this_month().strftime('%m/%d/%Y %H:%M:%S')
        security_event = {
            "type": event_type,
            "date": event_date
        }
        security_events.append(security_event)

    security_data = {
        "firewall_rules": firewall_rules,
        "security_events": security_events
    }
    return security_data

# Generate 1000 server log entries
server_logs = []
for _ in range(1000):
    server_id = fake.uuid4()
    server_ip = fake.ipv4()
    server_mac = fake.mac_address()
    server_location = fake.city()
    server_memory_used = random.randint(0, 100)
    server_temperature = random.randint(20, 85)
    server_os = random.choice(['RHEL', 'Fedora', 'FreeBSD', 'Windows Server 2003', 'Windows Server 2008', 'Windows Server 2012', 'Windows Server 2016', 'Windows Server 2019', 'Ubuntu', 'Debian', 'CentOS', 'SUSE', 'openSUSE', 'Arch Linux', 'Gentoo', 'Slackware', 'Linux Mint', 'Elementary OS', 'Kali Linux'])
    server_start_time = fake.date_time_this_decade().strftime('%m/%d/%Y %H:%M:%S')
    hardware_data = generate_hardware_data()
    security_data = generate_security_data()

    server_log = {
        "server_id": server_id,
        "server_ip": server_ip,
        "server_mac": server_mac,
        "server_location": server_location,
        "server_memory_used": server_memory_used,
        "server_temperature": server_temperature,
        "server_os": server_os,
        "server_start_time": server_start_time,
        "hardware": hardware_data,
        "security": security_data
    }
    server_logs.append(server_log)

# Write server log entries to a JSON file
with open('server-logs.json', 'w') as file:
    json.dump(server_logs, file, indent=4)
```

## Send data to OpenSearch

Now that you have the data on `server-logs.json` file, send the data to your Aiven for OpenSearch service using the following code:

```python
from opensearchpy import OpenSearch
import json

# OpenSearch configuration
opensearch_uri = 'YOUR OPENSEARCH URI GOES HERE'  # Replace with your OpenSearch server URI
index_name = "server-logs"

# Establish connection to OpenSearch
client = OpenSearch(opensearch_uri)

# Load JSON data to OpenSearch index
def load_json_to_opensearch(index_name, json_file):
    with open(json_file) as file:
        data = json.load(file)
    
    for doc in data:
        client.index(index=index_name, body=doc)

# Usage
load_json_to_opensearch(index_name, 'server-logs.json')
```

# Exercises

## Search servers by location

```python
from opensearchpy import OpenSearch

# OpenSearch configuration
opensearch_uri = 'YOUR OPENSEARCH URI GOES HERE'  # Replace with your OpenSearch server URI
index_name = 'server-logs'  # Replace with the desired index name

# Establish connection to OpenSearch
opensearch = OpenSearch(opensearch_uri)

# Search servers by location
def search_servers_by_location(location):
    query = {
        "query": {
            "match": {
                "server_location": location
            }
        }
    }
    response = opensearch.search(index='server-logs', body=query)
    return response['hits']['hits']

# Usage
servers = search_servers_by_location('New York')
for server in servers:
    print(server['_source'])
```

When I run this program, I see the following response:

```
{'server_id': 'de9e1e13-acf0-4ed4-bf8c-7a8b45a3ab98', 'server_ip': '185.155.164.108', 'server_mac': 'e1:69:6e:e3:c1:a2', 'server_location': **'New Susanport'**, ...}
{'server_id': 'dabe6dc5-6405-4426-b0bc-770c9f51d8b0', 'server_ip': '115.211.221.18', 'server_mac': '95:6c:04:ba:ce:50', 'server_location': **'New Dannymouth'**, ...}
{'server_id': '5e04c059-3fe6-4015-89e1-3f45d2aaf151', 'server_ip': '79.221.73.114', 'server_mac': '6a:91:8c:62:b2:ea', 'server_location': **'New Rachelville'**, ...}
{'server_id': '32afee58-0869-4e5c-94a7-5ad0883acf96', 'server_ip': '152.232.237.252', 'server_mac': '10:c9:e1:28:ae:9c', 'server_location': **'New Patricia'**, ...}
{'server_id': '8d52388f-60d6-4399-a1d2-8d085046a7b9', 'server_ip': '57.102.109.47', 'server_mac': '45:a4:8c:05:0c:26', 'server_location': **'New Jamietown'**, ...}
{'server_id': '5a037c0b-f050-4a12-8ad7-a4995624a523', 'server_ip': '34.15.24.173', 'server_mac': 'd1:46:c2:5f:32:18', 'server_location': **'New Josephport'**, ...}
```

## Server high memory usage

```python
from opensearchpy import OpenSearch

# Define the OpenSearch configuration
opensearch_uri = 'YOUR OPENSEARCH URI GOES HERE'  # Replace with your OpenSearch server URI

# Establish connection to OpenSearch
client = OpenSearch(opensearch_uri)

# Define the index name
index_name = 'YOUR OPENSEARCH INDEX NAME GOES HERE'

# Define the search query
query = {
    "query": {
        "range": {
            "server_memory_used": {
                "gt": 80
            }
        }
    }
}

# Execute the search query
response = client.search(index=index_name, body=query)

# Process the search results
hits = response['hits']['hits']
for hit in hits:
    source = hit['_source']
    server_id = source['server_id']
    server_memory_used = source['server_memory_used']
    server_os = source['server_os']
    # ... Retrieve and process other fields as needed
    print(f"Server ID: {server_id}, Memory Used: {server_memory_used}, OS: {server_os}")
```

The default behavior is to display up to 10 search results. If you want to display more than 10 results, you can modify the "size" parameter in the search query.

```python
