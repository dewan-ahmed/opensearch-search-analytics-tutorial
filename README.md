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

# Goal

In this tutorial, our goal is to explore OpenSearch, add some data, and perform various searches and analysis on the data. OpenSearch administration including OpenSearch high-availability, security, ACLs, etc are out of scope for this tutorial.

# Modules

[Prerequisites and setup](00_prereq_and_setup.md)

[Generate mock data](01_generate_mock_data.md)

[Send data to OpenSearch](02_send_data_to_opensearch.md)

[Search servers by location](03_search_servers_by_location.md)

[Find servers with high memory usage](04_find_servers_with_high_memory_usage.md)

[Find most used operating system](05_find_most_used_os.md)