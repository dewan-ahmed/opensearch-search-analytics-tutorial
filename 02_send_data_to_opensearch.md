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
