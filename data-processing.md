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
