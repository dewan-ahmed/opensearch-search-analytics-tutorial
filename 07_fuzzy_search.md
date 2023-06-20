## Fuzzy search

Fuzzy search is a technique used in information retrieval to find results that are similar or closely related to a given search query, even if they don't match exactly. It's designed to handle situations where there might be spelling mistakes, typos, or slight variations in the search terms.

In a fuzzy search, the search engine analyzes the search query and compares it to the indexed data using algorithms that take into account similarities and differences between characters or words. Instead of requiring an exact match, fuzzy search calculates a similarity score for each document in the index based on how closely it matches the query.

For example, if you perform a fuzzy search for the word "apple," the search engine may return results that include variations like "apples," "aplle," or even "aple," based on the configured level of fuzziness.

Fuzzy search is useful in scenarios where you want to account for potential mistakes or variations in user input and still provide relevant results. It can improve the search experience by capturing similar terms and increasing the chances of finding what the user is looking for, even if the exact term is not used.

For the following exercise, add this document containing `Toronto` as a server location:

```json
[
    {
        "server_id": "9xx55e05-1f42-437a-x5bb-bf888a52ff29",
        "server_ip": "52.237.120.133",
        "server_mac": "5e:08:ce:17:ef:09",
        "server_location": "Toronto",
        "server_memory_used": 69,
        "server_temperature": -26,
        "server_os": "RHEL",
        "server_start_time": "05/07/2021 23:40:11",
        "hardware": {
            "type": "Memory",
            "capacity": 10,
            "vendor": "Price, Taylor and Haynes"
        },
        "security": {
            "firewall_rules": [
                {
                    "port": 17189,
                    "protocol": "UDP",
                    "access": "ALLOW"
                },
                {
                    "port": 5795,
                    "protocol": "UDP",
                    "access": "ALLOW"
                },
                {
                    "port": 23282,
                    "protocol": "UDP",
                    "access": "DENY"
                },
                {
                    "port": 10647,
                    "protocol": "TCP",
                    "access": "ALLOW"
                },
                {
                    "port": 44683,
                    "protocol": "UDP",
                    "access": "ALLOW"
                }
            ],
            "security_events": [
                {
                    "type": "Login Attempt",
                    "date": "06/12/2021 18:50:33"
                }
            ]
        }
    }
]
```

Let's use OpenSearch's fuzzy search capabilities and have mis-spelled server location in the query:

```python
from opensearchpy import OpenSearch

# OpenSearch configuration
opensearch_uri = 'YOUR OPENSEARCH URI GOES HERE'  # Replace with your OpenSearch server URI
index_name = 'server-logs'  # Replace with the desired index name

# Establish connection to OpenSearch
opensearch = OpenSearch(opensearch_uri)

# Perform fuzzy search for servers
def perform_fuzzy_search(server_location):
    query = {
        "query": {
            "fuzzy": {
                "server_location": {
                    "value": server_location,
                    "fuzziness": "AUTO"
                }
            }
        }
    }
    response = opensearch.search(index='server-logs', body=query, size=10)
    return response['hits']['hits']

# Observe the typo Torono
servers = perform_fuzzy_search('Torono')
for server in servers:
    print(server['_source'])
```