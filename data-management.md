# Objectives

* Define an index that satisfies a given set of requirements
* Define and use an index template for a given pattern that satisfies a given set of requirements
* Define and use a dynamic template that satisfies a given set of requirements
* Define an Index Lifecycle Management policy for a time-series index
* Define an index template that creates a new data stream

# Define and index

[Index REST API](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/indices.html).

## Create an index

Create the following indices. Each index should be created with 1 primary and 0 replica shards:
* leads_b2b-000001
* leads_b2c-000001
* sales_b2c-000001

<solution>
```
PUT /leads_b2b-000001
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}

PUT /leads_b2c-000001
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}

PUT /sales_b2c-000001
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```
</solution>

## Get index settings



## Update an index



# Define and use an index template

# Define and use a dynamic template

Define an dynamic template that for all strings ending with `_es` assigns a special `spanish` analyzer. For the remaining strings, apply `english` analyzer.

<solution>

```


```

</solution>

----




# Define an Index Lifecycle Management policy for a time-series index

# Define an index template that creates a new data stream

# Other actions

## Delete an index

## Delete multiple indexes

## Delete all indexes

