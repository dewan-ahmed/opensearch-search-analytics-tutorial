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

Execute the program and save the output to a file for easier visual rather than seeing a large response in terminal:

```shell
python3 search_servers_by_location.py > servers_by_location
```

When I run this program, I see the following response:

```
{'server_id': 'de9e1e13-acf0-4ed4-bf8c-7a8b45a3ab98', ..., 'server_location': 'New Susanport', ...}
{'server_id': 'dabe6dc5-6405-4426-b0bc-770c9f51d8b0', ..., 'server_location': 'New Dannymouth', ...}
{'server_id': '5e04c059-3fe6-4015-89e1-3f45d2aaf151', ..., 'server_location': 'New Rachelville', ...}
{'server_id': '32afee58-0869-4e5c-94a7-5ad0883acf96', ..., 'server_location': 'New Patricia', ...}
{'server_id': '8d52388f-60d6-4399-a1d2-8d085046a7b9', ..., 'server_location': 'New Jamietown', ...}
{'server_id': '5a037c0b-f050-4a12-8ad7-a4995624a523', ..., 'server_location': 'New Josephport', ...}
```

That's not what I expected :thinking: I was looking for servers in **New York**. What happend?

The behavior you're observing is due to the default text analysis process performed by OpenSearch when indexing and searching textual data. OpenSearch uses analyzers to process text, which include tokenization, normalization, and other operations to generate terms that are used for indexing and searching.

By default, OpenSearch uses the standard analyzer, which performs tokenization based on whitespace and punctuation. It treats the input text as a sequence of terms and indexes them accordingly. For example, when indexing the location "New York," the standard analyzer will tokenize it into two terms: "New" and "York."

During the search process, when you query for "New York," the same standard analyzer is applied to the query, resulting in the query terms "New" and "York." OpenSearch then matches these terms against the indexed terms.

If you're getting results like "New Susanport" or "New Dannymouth," it suggests that those locations were also present in the indexed data and they contain the term "New." The search process matches the query term "New" with these indexed terms and returns the corresponding results.

Let's use an updated version of the program for searching servers by location:

```python
...
# Only update the query match part of the program
search_query = {
    "query": {
        "match": {
            "server_location.keyword": "New York"
        }
    }
}
...
```

In this updated version, we are using the `server_location.keyword` field for the search, which is a keyword field. Keyword fields are not analyzed, meaning they are indexed as a single term without any tokenization or normalization. This allows us to perform an exact match on the location field.

By using the `.keyword` suffix in the field name (`server_location.keyword`), we are referencing the exact keyword version of the field in the search query.

When you run this program with the search query "New York," it will only return the servers that have an exact match with the location "New York" in the `server_location` field. This avoids matching partial terms like "New Susanport" or "New Dannymouth" that might exist due to the default standard analyzer.

The program will iterate over the search results and print the server ID and location for each matching server in New York.
