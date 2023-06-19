## Find most used operating system

```python
from opensearchpy import OpenSearch

# OpenSearch configuration
opensearch_uri = 'YOUR OPENSEARCH URI GOES HERE'  # Replace with your OpenSearch server URI
index_name = 'server_logs'  # Replace with the desired index name

# Establish connection to OpenSearch
opensearch = OpenSearch(opensearch_uri)

# Get most used operating systems
def get_most_used_operating_systems(top_n):
    query = {
        "aggs": {
            "top_os": {
                "terms": {
                    "field": "server_os.keyword",
                    "size": top_n
                }
            }
        }
    }
    response = opensearch.search(index='server-logs', body=query)
    buckets = response['aggregations']['top_os']['buckets']
    return [(bucket['key'], bucket['doc_count']) for bucket in buckets]

# Usage
top_os = get_most_used_operating_systems(3)
print("Top Operating Systems:")
for os, count in top_os:
    print(f"OS: {os}, Count: {count}")
```

[Aggregations](https://opensearch.org/docs/latest/aggregations/) in OpenSearch allow you to perform advanced analytics on your data by grouping, filtering, and computing metrics on the aggregated groups. They enable you to gain insights and extract meaningful information from your dataset. Aggregations are commonly used for various analytics tasks, including computing metrics, creating histograms, date histograms, and more, based on specific fields or combinations of fields in your documents.

In the case of the program that finds the most used operating system, it utilizes the "terms" aggregation, which is used to group the server logs based on the values of the "server_os" field. Here's how the aggregation works:

1. The program constructs an aggregation query specifying the "terms" aggregation on the "server_os" field.
2. When the query is executed, OpenSearch scans through the server logs and creates buckets for each unique value of the "server_os" field.
3. Each bucket represents a distinct operating system value, and OpenSearch assigns the relevant server logs to their respective buckets.
4. OpenSearch also counts the number of logs within each bucket, which corresponds to the document count.
5. Once the aggregation is completed, OpenSearch returns the aggregation results, which include the buckets with the operating system terms and their respective document counts.

By examining the aggregation results, you can identify the operating system term(s) that appear most frequently in the server logs. In the program, the operating system with the highest document count is considered the most used operating system.
