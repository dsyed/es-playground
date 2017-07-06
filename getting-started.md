# Getting Started

A quick guide to using the basic functions of Elasticsearch 5.4.

Mostly stolen wholesale from Elastic's own [Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html) page.

## List all indices
```
GET _cat/indices?h=i
```

## Create an index
```
PUT legends?pretty
GET _cat/indices?h=i
```

## Index a document
```json
PUT legends/jpl/1?pretty
{
    "name": "Randy Moss"
}
```

## Retrieve a document
```
GET legends/jpl/1?pretty
```

## Delete an index
```
DELETE legends?pretty
GET _cat/indices?h=i
```

## Replace a document
```json
PUT legends/jpl/1?pretty
{
    "name": "Randy Moss"
}

PUT legends/jpl/1?pretty
{
    "name": "Jeff Liu"
}
```

## Index a document without specifying an ID
```json
PUT legends/jpl?pretty
{
    "name": "Randy Moss"
}
```

## Update a document
```json
POST legends/jpl/1/_update?pretty
{
    "doc": { "name": "Randal Moss" }
}

POST legends/jpl/1/_update?pretty
{
    "doc": { "name": "Randal Moss", "bootcamps": 2 }
}

POST legends/jpl/1/_update?pretty
{
    "script": "ctx._source.bootcamps += 1"
}
```

## Delete a document
```
DELETE legends/jpl/2?pretty
GET legends/_search
```

## Bulk index a data file
Download this [sample dataset](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true). Save it to your current directory as `accounts.json`.

In your terminal (replace `localhost` with your ES host):
```bash
curl -H "Content-Type: application/json" -XPOST -u elastic:changeme 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@accounts.json"
curl -u elastic:changeme 'localhost:9200/_cat/indices?h=i'
```

## Get all documents
```json
GET bank/_search?q=*&sort=account_number:asc&pretty

GET bank/_search
{
    "query": { "match_all": {} },
    "sort": [
        { "account_number": "asc" }
    ]
}
```

## Get documents 11-20
```json
GET bank/_search
{
    "query": { "match_all": {} },
    "from": 10,
    "size": 10
}
```

## What does this do?
```json
GET bank/_search
{
    "query": { "match_all": {} },
    "sort": { "balance": { "order": "desc" } }
}
```

## Get specific fields
```json
GET bank/_search
{
    "query": { "match_all": {} },
    "_source": ["account_number", "balance"]
}
```

## Basic searching
```json
GET bank/_search
{
    "query": { "match": { "address": "mill" } }
}

GET bank/_search
{
    "query": { "match": { "address": "mill lane" } }
}

GET bank/_search
{
    "query": { "match_phrase": { "address": "mill lane" } }
}
```

## `must` query
```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

## `should` query
```json
GET bank/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

## `must_not` query
```json
GET bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

## What does this do?
```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

## Basic filtering
```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

## Group by state
```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

## Account balance by state
```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

## Account balance by state, sorted by avg. balance
```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

## What does this do?
```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```