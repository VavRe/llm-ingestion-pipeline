# Scalable LLM-Based News QA System 

> **Note**: This Repository is a work in progress. The code is being cleaned and organized for better accessibility.


This repository contains a scalable, end-to-end news Q&A system powered by Large Language Models (LLMs), specifically designed for efficient retrieval of LLM-related news. The system uses a robust data ingestion pipeline, a Retrieval Augmented Generation (RAG) based QA model, and a caching layer for optimized performance, all orchestrated using Docker Compose.

## System Overview

The system operates through the following key components:

1. **Crawler**: Crawls and gathers the latest LLM-related news articles from various online sources.

2. **Content Conversion**:  Converts crawled HTML pages into Markdown format using Jina AI's Reader-LM.

3. **Markdown-Aware Chunking**: Divides the Markdown content into semantically meaningful chunks.

4. **Embedding Generation & Storage**: Embeds these chunks using the `stella_en_400M_v5` embedder and stores them in Milvus.

5. **Asynchronous Query Generation (via NATs Queue)**: After chunks are embedded, their IDs are published to a NATs queue. A subscriber monitors this queue every 10 minutes, generating queries using a fine-tuned LLaMa 3.2 (3B) model. These generated queries are also embedded and stored in Milvus.

6. **RAG-based QA with Milvus & LLM**: Upon receiving a user query, the system calculates its similarity with the generated queries in Milvus. The most similar generated queries are retrieved, and their associated document chunks are used as context for the LLaMa LLM to generate a comprehensive answer.

7. **Redis Caching with Invalidation**:   The query and answer pairs are cached in Redis for faster retrieval.

    - **Cache Hit**: If a new user query's similarity exceeds a predefined threshold with a cached query, the cached answer is returned directly.
    - **Cache Invalidation**: The cache is invalidated if newly generated queries (from freshly crawled content) are highly similar to cached queries. This ensures that answers are generated from the most recent and relevant information. The answer is then regenerated with the new context and cached.


## Repository Structure

```
├── src
│   ├── crawler.py
│   ├── content_converter.py
│   ├── chunker.py
│   ├── embedder.py
│   ├── query_generator.py
│   ├── nats_publisher.py
│   ├── nats_subscriber.py
│   ├── rag_qa.py
│   ├── cache.py
│   └── main.py
├── configs
│   ├── crawler_config.yaml
│   ├── system_config.yaml
│   ├── nats_config.yaml  
│   └── llama_config.yaml
├── docker-compose.yml
├── .dockerignore
└── requirements.txt
└── README.md
```


## Getting Started

1. **Prerequisites**: Ensure you have Docker and Docker Compose installed.

2. **Build Images**: Navigate to the root directory and run `docker-compose build` to build the necessary Docker images.

3. **Start Services**: Run `docker-compose up -d` to start all the services (crawler, converter, chunker, embedder, query generator, RAG QA, NATs, Milvus, and Redis) in detached mode.  The `-d` flag runs the containers in the background.

4. **Access Services**: You can then interact with the services according to their exposed ports as defined in the `docker-compose.yml` file.

5. **Stop Services**:  Use `docker-compose down` to stop and remove the containers, networks, and volumes.  Add the `-v` flag (`docker-compose down -v`) to also remove the data volumes if you want to completely reset the system.





## Key Features

* **Scalable**: Handles large news data volumes.
* **Efficient Retrieval**: Milvus and Redis provide fast search and retrieval.
* **Real-time Updates**: NATs and cache invalidation ensure up-to-date information.
* **Modular Design**: Flexible and easily modifiable components.
* **Dockerized**: Simplified deployment and management with Docker Compose.



