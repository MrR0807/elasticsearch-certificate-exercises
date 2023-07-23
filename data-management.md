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

<details>

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

</details>

## Get index settings

<details>

```
GET /sales_b2c-000001
GET /sales_b2c-000001/_settings
GET /sales_b2c-000001/_mapping
GET /sales_b2c-000001/_alias
```

</details>


## Update an index

Increase the replica count to 1.

<details>

```
PUT /leads_b2b-000001/_settings
{
  "settings": {
    "number_of_replicas": 1
  }
}
```

</details>


# Define and use an index template

## Index Template

```
POST _index_template/template_1
{
  "index_patterns": ["*cars*"],
  "priority": 20,
  "template": {
    "mappings": {
      "properties": {
        "created_at": {
          "type": "date"
        },
        "created_by": {
          "type": "text"
        }
      }
    }
  }
}
```

You need to create the following index templates in Elasticsearch to properly format the telemetry data indices:
* `load` for indices that start with "load-"
* `cpu` for indices that start with "cpu-"
* `memory` for indices that start with "memory-"

Since this is a single-node cluster, you will need to configure the index templates to create all indices with 1 primary and 0 replica shards to ensure the indices that use these templates are allocated with a green state.

```
PUT _index_template/load_template
{
  "index_patterns": ["load-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}

PUT _index_template/cpu_template
{
  "index_patterns": ["cpu-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}

PUT _index_template/memory_template
{
  "index_patterns": ["memory-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}
```

## Component Template

Create a component template and reuse it in previous exercise.

```
PUT _component_template/dev-env
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}
```

```
POST _index_template/memory_template
{
  "index_patterns": ["memory-*"],
  "composed_of":["dev-env"]
}
```

Simulate index API

```
POST /_index_template/_simulate_index/memory-something
```

---

Create an index with default `english` analyzer.

```



```


# Define and use a dynamic template

Define an dynamic template that for all strings ending with `_es` assigns a special `spanish` analyzer. For the remaining strings, apply `english` analyzer.

<details>

```


```

</details>

----




# Define an Index Lifecycle Management policy for a time-series index

# Define an index template that creates a new data stream

# Other actions

## Delete an index

## Delete multiple indexes

## Delete all indexes

