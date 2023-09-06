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

You need to create the following index templates in Elasticsearch to properly format the telemetry data indices:
* `load` for indices that start with "load-"
* `cpu` for indices that start with "cpu-"
* `memory` for indices that start with "memory-"

Since this is a single-node cluster, you will need to configure the index templates to create all indices with 1 primary and 0 replica shards to ensure the indices that use these templates are allocated with a green state.

<details>

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

</details>

------

You need to create an index template for index containing word *cars*. It should have priority 20, and defaults fields of *created_at* and *created by*.

<details>
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
</details>

## Component Template

Create a component template and reuse it in previous exercise.

<details>
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
</details>

---

# Define and use a dynamic template

Define a dynamic template, where all fields starting with *ip* name would be mapped to **ip** type.

<details>

```
PUT leads-1
{
  "mappings": {
    "dynamic": "true",
    "dynamic_templates": [
      {
        "strings_as_ip": {
          "match_mapping_type": "string",
          "match": "ip*",
          "mapping": {
            "type": "ip"
          }
        }
      }
    ]
  }
}
```
</details>

---

Define a dynamic template that changes all new added fields starting with `ip_` to type `ip`. Also, all strings ending with `es` having `spanish` analyzer and all strings ending with `_en` - `english` analyzer.

<details>
```
PUT my-index
{
  "mappings": {
    "properties": {
      "id_long": {
        "type": "long"
      },
      "name": {
        "type": "text"
      }
    },
    "dynamic_templates": [
      {
        "ip-strings-to-ip-type": {
          "match_mapping_type": "string",
          "match": "ip_*",
          "mapping": {
            "type": "ip"
          }
        }
      },
      {
        "es-analyzer": {
          "match_mapping_type": "string",
          "match": "*_es",
          "mapping": {
            "analyzer": "spanish"
          }
        }
      },
      {
        "en-analyzer": {
          "match_mapping_type": "string",
          "match": "*_en",
          "mapping": {
            "analyzer": "english"
          }
        }
      }
    ]
  }
}
```
</details>
----

# Define an Index Lifecycle Management policy for a time-series index

Trying to do with date time math fails due to Elastic bug: https://github.com/elastic/kibana/issues/128701.


Short reminder.
1. Create a lifecycle policy that defines the appropriate phases and actions. See [Create a lifecycle policy above](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/getting-started-index-lifecycle-management.html#manage-time-series-data-without-data-streams:~:text=and%20actions.%20See-,Create%20a%20lifecycle%20policy,-above).
2. [Create an index template](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/getting-started-index-lifecycle-management.html#manage-time-series-data-without-data-streams:~:text=lifecycle%20policy%20above.-,Create%20an%20index%20template,-to%20apply%20the) to apply the policy to each new index.
3. [Bootstrap an index](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/getting-started-index-lifecycle-management.html#ilm-gs-alias-bootstrap) as the initial write index.
4. [Verify indices are moving through the lifecycle phases](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/getting-started-index-lifecycle-management.html#ilm-gs-alias-check-progress) as expected.


---

You are a cyber security analyst who is in charge of collecting and maintaining an audit log of user access changes at your company. For this, you will use Elasticsearch to store the audit data in a daily index with the format audit-MM-DD-YYY. However, you need to first configure an index lifecycle management (ILM) policy for the audit data as follows:
* Data is first ingested as hot data.
* After 7 Days, the data is converted to cold data with a readonly index.
* After 365 days, the data is deleted.

Once the ILM policy is created, you need to create the audit_template index template to match any index with the naming pattern of audit-, and apply the audit_policy ILM policy. Also, because you are working on a single-node cluster, the audit_template index template should be configured to create indices with 1 primary and 0 replica shards in order to maintain a green cluster state. Lastly, create and verify the first audit index with the current date using the audit-MM-DD-YYY format.

<details>

```
PUT _ilm/policy/audit_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {}
      },
      "cold": {
        "min_age": "7d",
        "actions": {
         "readonly" :{}
        }
      },
      "delete": {
        "min_age": "365d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}

PUT _index_template/audit-template
{
  "index_patterns": ["audit-*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "audit_policy",      
      "index.lifecycle.rollover_alias": "audit"
    }
  }
}

PUT audit-09-28-2023
GET audit-*/_ilm/explain
```
</details>

---

For example, if you are indexing metrics data from a fleet of ATMs into Elasticsearch, you might define a policy that says:
* When the total size of the index’s primary shards reaches 50GB, roll over to a new index.
* Move the old index into the warm phase, mark it read only, and shrink it down to a single shard.
* After 7 days, move the index into the cold phase and move it to less expensive hardware.
* Delete the index once the required 30 day retention period is reached.

```


```

# Define an index template that creates a new data stream

Define a new data stream, which rolls over to new index when index size reaches 1 document. Cold storage should be run after 1d. And deletion phase after 365 days.

<details>

```
PUT _ilm/policy/my-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_docs": 1
          }
        }
      },
      "cold": {
        "min_age": "1d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          }
        }
      },
      "delete": {
        "min_age": "365d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}

PUT _index_template/my-index-template
{
  "index_patterns": ["audit-*"],
  "data_stream": {},
  "template": {
     "settings": {
       "index.lifecycle.name": "my-policy"
     }
  }
}

PUT _data_stream/audit-something
GET _data_stream/audit-something

POST audit-something/_doc
{
  "hello": "world3",
  "@timestamp": "2099-07-06T16:21:15.000Z"
}
```
</details>


# Other actions

## Delete an index

## Delete multiple indexes

## Delete all indexes

