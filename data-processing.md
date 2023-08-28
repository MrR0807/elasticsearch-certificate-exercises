# Objectives

* Define a mapping that satisfies a given set of requirements
* Define and use a custom analyzer that satisfies a given set of requirements
* Define and use multi-fields with different data types and/or analyzers
* Use the Reindex API and Update By Query API to reindex and/or update documents
* Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents
* Define runtime fields to retrieve custom values using Painless scripting


Test an analyzer

```
GET _analyze
{
  "text": "James Bond 007",
  "analyzer": "simple"
}
```

---

Create an index with default `english` analyzer.

```
PUT exercise-1
{
  "mappings": {
    "properties": {
      "id": {
        "type": "long"
      },
      "name": {
        "type": "text"
      }
    }
  },
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "english"
        }
      }
    }
  }
}
```

---

Define an dynamic template that for all strings ending with `_es` assigns a special `spanish` analyzer. For the remaining strings, apply `english` analyzer.

<details>

```


```

</details>


---

Get index mapping

```
GET my-index/_mapping
```
