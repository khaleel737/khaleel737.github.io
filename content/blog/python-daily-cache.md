+++
date = "2024-08-27"
title = "@daily_cache implementation in Python"
tags = ['python']
+++

I spend a lot of time at Carbonfact working on datasets shared by our customers. We typically set things up so that our customers can export data automatically. They usually deposit files to a GCP bucket, with a script, once a day. We then have an ETL script for each customer that runs afterwards to fetch their latest data and process it.

During development, I load customer data to my laptop and work on it. The datasets can be quite heavy, and it takes time to fetch them, so I cache them to save some time. Python has [something](https://docs.python.org/3/library/functools.html) for this in its standard library:

```py
import functools

@functools.cache
def load_raw_customer_data():
    # Slow I/O...
    ...

def process_customer_data():
    raw = raw_customer_data()
    # Many operations, much compute, but it's fast...
```

Under the hood, `@functools.cache` caches function outputs in RAM. The issue I have is that if I turn my computer off for lunch, then I lose the cache when I come back to it in the afternoon. Likewise if I reload my Jupyter notebook. But in fact, the cache is still valid because the customer hasn't uploaded any new data -- because I know they do it once a day.

A simple solution is to use a cache that persists the data to disk. There is a Python package called joblib that [does this](https://joblib.readthedocs.io/en/latest/memory.html#memory). Here's an example:

```py
import joblib

memory = joblib.Memory(
    location='~/customer_cache',
    verbose=0
)

@memory.cache
def load_raw_customer_data():
    # Slow I/O...
    ...
```

The only issue with the snippet above is that the cache is never invalidated. There is actually a [documented way](https://joblib.readthedocs.io/en/latest/memory.html#custom-cache-validation) to empty the cache after a certain amount of time, but that's not exactly what I want. I want to empty the cache once a day. For instance, if the last time I retrieved the data was yesterday, then I want to re-fetch it, even if it was only a few hours ago.

Thankfully, joblib allows you to pass a custom cache validation callback. You can provide a function, which takes as argument a metadata dictionary. One of the fields in this dictionary is called `time`, and represents the last moment when the function was called with a cache hit. We can therefore compare this field to the current date, and decide whether the cache should hit or miss. Here's how to do it:

```py
import joblib

memory = joblib.Memory(
    location='~/customer_cache',
    verbose=0
)

def daily_cache_validation_callback(metadata):
    last_call_at = dt.datetime.fromtimestamp(metadata['time'])
    return last_call_at.date() == dt.date.today()

daily_cache = memory.cache(
    cache_validation_callback=daily_cache_validation_callback
)

@daily_cache
def load_raw_customer_data():
    # Slow I/O...
    ...
```

It's quite straightforward to unit test the `@daily_cache` decorator thanks to the [FreezeGun](https://github.com/spulec/freezegun) package:

```py
from freezegun import freeze_time

# Clear the cache for reproducibility
memory.clear(warn=False)

@daily_cache
def load_raw_customer_data():
    print(f"Loading data from scratch because it's {dt.datetime.now()}")

with freeze_time("2024-08-27 09:00:00"):
    load_raw_customer_data()

with freeze_time("2024-08-27 15:00:00"):
    load_raw_customer_data()

with freeze_time("2024-08-28 06:00:00"):
    load_raw_customer_data()
```

```
Loading data from scratch because it's 2024-08-27 09:00:00
Loading data from scratch because it's 2024-08-28 06:00:00
```

This is exactly what I needed. I hope it can be helpful to other people too.

There is however one last thing that isn't perfect with this solution. Sometimes, we notify customers that something is wrong in the data they shared, and occasionally they fix the issue and re-upload the data in the same day. In this case, the cache will still hit, and we will process the old data. It's a bit error-prone to have to clear the cache manually.

In an ideal world, I shouldn't have to assume files are refreshed every day, and the caching mechanism should figure out if something has changed or not. This usually by checking the last modification date of the remote file. This implies access to a file system, which can be tricky to abstract when your customers use a mix of AWS S3, GCP buckets, and other cloud storage solutions. The main advantage of the `@daily_cache` decorator above is that it agnostic of the data source, and so it works whatever file it is you are loading.

There is however a promising solution called [Filesystem Spec](https://filesystem-spec.readthedocs.io/en/latest/index.html) (fsspec for short). It's a Python package that provides a common interface to many file systems, including cloud storage solutions. There is even [some documentation](https://filesystem-spec.readthedocs.io/en/latest/features.html#caching-files-locally) on local caching. It isn't clear how to exactly set it up so that files are only re-downloaded when they change, but ChatGPT [figured it out](https://chatgpt.com/share/c6238444-99aa-4c77-972d-3f48b50fb942).
