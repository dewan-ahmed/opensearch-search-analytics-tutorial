## Find servers by location

The `multi_match` part in the following program is a type of query used in OpenSearch to search for a specific keyword across multiple fields in the index. It allows you to search for the keyword in different fields simultaneously and retrieves documents that contain a match in any of those fields. This allows for a more comprehensive search, as it considers multiple fields to find relevant information related to the keyword.

But first, let's add the following document to `server-logs` index:

```json
[
    {
        "server_id": "ac14c080-97d7-4af5-9777-edcabdaec98d",
        "server_ip": "53.93.117.01",
        "server_mac": "ad:32:88:0c:f0:be",
        "server_location": "Kansas City",
        "server_memory_used": 69,
        "server_temperature": 26,
        "server_os": "RHEL",
        "server_start_time": "05/07/2020 23:40:11",
        "hardware": {
            "type": "Memory",
            "capacity": 10,
            "vendor": "KCDC - Kansas City"
        },
        "security": {
            "firewall_rules": [
                {
                    "port": 17189,
                    "protocol": "TCP",
                    "access": "ALLOW"
                },
                {
                    "port": 5795,
                    "protocol": "UDP",
                    "access": "DENY"
                },
                {
                    "port": 23281,
                    "protocol": "UDP",
                    "access": "DENY"
                },
                {
                    "port": 10647,
                    "protocol": "TCP",
                    "access": "DENY"
                },
                {
                    "port": 44683,
                    "protocol": "UDP",
                    "access": "ALLOW"
                }
            ],
            "security_events": [
                {
                    "type": "Login Attempt from Kansas City",
                    "date": "06/12/2019 18:50:33"
                }
            ]
        }
    }
]
```

Save the file as `additional_server_log.json` and use the same `load_json_to_opensearch` function from a previous section to send this additional document.

```python
...
load_json_to_opensearch('server-logs', 'additional_server_log.json')
...
```

The following code searches across server location, hardware vendor, and security events type of each document for a match on the given query.

```python
from opensearchpy import OpenSearch

# OpenSearch configuration
opensearch_uri = 'YOUR OPENSEARCH URI GOES HERE'  # Replace with your OpenSearch server URI
index_name = 'server-logs'  # Replace with the desired index name

# Establish connection to OpenSearch
opensearch = OpenSearch(opensearch_uri)

# Search servers by keyword
def search_servers_by_keyword(keyword):
    # Define the multi-match query
    search_query = {
        "query": {
            "multi_match": {
                "query": keyword,
                "fields": ["server_location", "hardware.vendor", "security.security_events.type"]
            }
        }
    }
    response = opensearch.search(index='server-logs', body=search_query, size=10)
    return response['hits']['hits']

# Usage
servers = search_servers_by_keyword('Kansas City')
for server in servers:
    print(server['_source'])
```

