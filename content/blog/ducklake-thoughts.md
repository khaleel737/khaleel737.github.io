+++
date = "2025-06-09"
title = 'Thoughts on DuckLake'
tags = ['data-eng']
+++

[DuckLake](https://ducklake.select/) is the new data lake/warehouse from the makers of DuckDB. I really like the direction they're taking. I'm hopeful it has the potential to streamline the data engineering workflow for many people, vastly reducing costs along the way.

I'm a bit of a nut and don't use SQLMesh or dbt. Instead, I built [lea](https://github.com/carbonfact/lea) a few years ago, and we still use it at [Carbonfact](https://carbonfact.org). I would probably pick SQLMesh if I had to start over, but lea allows me to explore new ideas, so I'm sticking to it for now.

I just added support in lea for DuckLake, which wasn't too hard because DuckDB is already supported. Here's an example to run some analytical queries for another [project](https://github.com/MaxHalford/bike-sharing-history) of mine:

```sh
git clone https://github.com/MaxHalford/bike-sharing-history --depth 1
cd bike-sharing-history
echo "
LEA_WAREHOUSE=ducklake
LEA_DUCKLAKE_DATA_PATH=gcs://bike-sharing-analytics
LEA_DUCKLAKE_CATALOG_DATABASE=metadata.ducklake
LEA_DUCKLAKE_S3_ENDPOINT=storage.googleapis.com
" > .env
uvx --from lea-cli lea run --scripts analytics
```

*I do recommend [uvx](https://docs.astral.sh/uv/guides/tools/)*

So what's the point of DuckLake? In my opinion, the technical details of how it handles metadata versus, say, Iceberg, are not that important. It might be the case that Iceberg doesn't cope well with many writes and small files, but that's [already](https://www.theregister.com/2025/06/05/ducklake_db_industry_reacts/) being worked on. Most of the technical details go over my head anyway.

What excites me about DuckLake is that it can make DuckDB function as a data warehouse. DuckLake advertises key features like snapshotting, ACID transactions, and time travel, which are all great. But the most important feature is that it enables running DuckDB queries in a remote data lake, which is something that wasn't straightforward so far.

Hear me out. When you're running your dbt/SQLMesh/lea DAG with DuckDB:

1. The queries need to be executed on a node.
2. The query outputs need to be stored on a disk.

The "problem" with DuckDB is that everything happens locally. That's great for small projects, but it's going to kill your laptop for anything semi-serious. The advantage of using Snowflake/BigQuery is that compute and storage happen in the cloud, and your machine is just orchestrating the queries.

It can be ok to use your laptop to do the compute, especially for development. Today's personal machines are powerful. By my estimate, you have to spend ~$0.54/hour to get access to a cloud machine with the same specs as a 2024 MacBook Pro. But you don't want to be storing terabytes of intermediate results on your laptop.

When I write SQL queries as part of a DAG, my dream workflow is to:

1. Clone the production database in the cloud.
2. Edit the SQL queries.
3. Run the queries from my laptop, with the compute happening in the cloud.
4. Have the query outputs stored in the cloud.
5. Q/A the results from my laptop.
6. Push the changes to GitHub once I'm happy.
7. Have steps 3. and 4. run automatically in CI/CD.

This is what I do with BigQuery at work. It's fine. The only problem is that local development costs money, which feels wrong. I would love to transpile my SQL queries to DuckDB and run them locally. But in order to do that, I would have to clone the production database to my laptop, where there isn't enough disk space.

DuckDB supports the S3 API. Meaning that the data can live in `.parquet` files in an S3/GCS/R2 bucket. So we could imagine running compute locally, and having the query outputs stored in the bucket. Alas, DuckDB's S3 [write](https://duckdb.org/docs/stable/core_extensions/httpfs/s3api.html#writing) support is limited to copying a local table to S3. So you would need to write the query outputs locally, and then copy them to the bucket, which is not ideal.

This is why I like DuckLake. When you `ATTACH` to DuckLake, there's a `DATA_PATH` parameter with many [storage options](https://ducklake.select/docs/stable/duckdb/usage/choosing_storage) to choose from. This provides a way to execute DuckDB queries locally, and have the outputs stored in the cloud. I assume the query output chunks are progressively written to `.parquet` files in the bucket.

DuckLake is also quite simple to set up. With these environment variables:

```sh
LEA_WAREHOUSE=ducklake
LEA_DUCKLAKE_DATA_PATH=gcs://bike-sharing-analytics
LEA_DUCKLAKE_CATALOG_DATABASE=metadata.ducklake
LEA_DUCKLAKE_S3_ENDPOINT=storage.googleapis.com
```

Under the hood, lea runs the following SQL commands to set up the DuckLake connection:

```sql
ATTACH 'ducklake:metadata.ducklake' AS my_ducklake (
    DATA_PATH 'gcs://bike-sharing-analytics'
);
USE my_ducklake;
SET s3_endpoint='storage.googleapis.com'
```

What about MotherDuck? Well, the truth is that the same setup can be achieved with MotherDuck. Here are the differences I see:

- With DuckLake the data lives in your bucket, and you don't need to use a third-party service -- important for enterprise.
- MotherDuck can handle compute for you, which is an advantage. In particular, I'm not exactly sure how DuckLake is meant to be used from CI/CD pipelines, where you don't usually have access to a big machine.
- MotherDuck [doesn't](https://motherduck.com/docs/concepts/architecture-and-capabilities/#considerations-and-limitations) support all of DuckDB's features.
- MotherDuck gives you access to a nice UI, but I'm sure they'll add support for DuckLake soon.

*For the record, lea supports MotherDuck too:*

```sh
LEA_WAREHOUSE=motherduck
MOTHERDUCK_TOKEN=<get this from https://app.motherduck.com/settings/tokens>
LEA_MOTHERDUCK_DATABASE=bike_sharing
```

To summarize, the next big thing I want to see is the ability to transpile BigQuery/Snowflake SQL queries to DuckDB, have them run locally (or not), and have the outputs stored in a blob storage bucket. DuckLake is a step in the right direction.
