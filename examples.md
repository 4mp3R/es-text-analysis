## Text analysis

### Taking a look behind the scenes

Analyze text without storing it into a document:

```
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["lowercase"],
  "text": "Hello, world! :)",
  "explain": true
}
```

Check how the text stored in a document was analyzed:

```
POST myindex/_doc
{
  "text": "<a href=\"bla.com\">Click me nöw!</a>"
}

GET /myindex/_termvectors/cbLWSogBVqhxYr88n8Sk?fields=text
```

### Char filters

```
GET _analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    "html_strip"
  ],
  "text": "<a href=\"url\">Click me</a>"
}

GET _analyze
{
  "tokenizer": "keyword",
   "char_filter": [
    {
      "type": "mapping",
      "mappings": [
       ":) => happy",
       ":D => happy",
       ":( => sad",
       "D: => sad"
      ]
    }
  ],
  "text": "Sam feels :D today"
}

GET _analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    {
          "type": "pattern_replace",
          "pattern": "\\d{3}-(\\d{3})-\\d{3}",
          "replacement": "$1"
        }
  ],
  "text": "Order ID: 123-456-789"
}
```

### Tokenizers

```
GET _analyze
{
  "tokenizer": "whitespace",
  "text": "Hey there, what's up?"
}

GET _analyze
{
  "tokenizer": "standard",
  "text": "Hey there,what's up?"
}

GET _analyze
{
  "tokenizer": "letter",
  "text": "Hey there,what's up?"
}

GET _analyze
{
  "tokenizer": {
    "type": "edge_ngram",
    "min_gram": 2,
    "max_gram": 3,
    "token_chars": "letter"
  },
  "text": "Hello world!"
}

GET _analyze
{
  "tokenizer": "keyword",
  "text": "Hello world!"
}

GET _analyze
{
  "tokenizer": "path_hierarchy",
  "text": "/home/alex/docs"
}
```

### Token filters

```
GET _analyze
{
  "tokenizer" : "whitespace",
  "filter" : ["asciifolding"],
  "text" : "Menü: Açaí à la carte ①②③"
}

GET _analyze
{
  "tokenizer" : "standard",
  "filter" : ["elision"],
  "text" : "l'arbre"
}

GET _analyze
{
  "tokenizer" : "standard",
  "filter" : ["lowercase"],
  "text" : "TeSt TExT"
}

GET _analyze
{
  "tokenizer" : "standard",
  "filter" : ["uppercase"],
  "text" : "TeSt TExT"
}

GET _analyze
{
  "tokenizer" : "standard",
  "filter" : [{
    "type": "multiplexer",
    "filters": ["uppercase", "lowercase"]
  }],
  "text" : "TeSt"
}

GET _analyze
{
  "tokenizer" : "standard",
  "filter" : ["unique"],
  "text" : "the beauty and the beast"
}

GET _analyze
{
  "tokenizer": "standard",
  "filter": ["stemmer"],
  "text": "Low hanging fruits"
}

GET _analyze
{
  "tokenizer": "standard",
  "filter": [
    "stemmer",
    {
      "type": "synonym",
      "lenient": true,
      "synonyms": [
        "indistinguishable => identical",
        "sample => fragment"
      ]
    }
  ],
  "text": "The two samples are indistinguishable"
}
```

### Standard analyzers

```
GET _analyze
{
  "analyzer": "standard",
  "text": "Hello, what's up?"
}

GET _analyze
{
  "analyzer": "simple",
  "text": "Hello, what's up?"
}

GET _analyze
{
  "analyzer": "whitespace",
  "text": "Hello, what's up?"
}

GET _analyze
{
  "analyzer": "italian",
  "text": "C'è un'aquila nel cielo"
}
```

### Example analysis process

Create an index with a custom default text analyzer:

```
PUT myindex
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  }
}
```

Analyze the text:

```
GET myindex/_analyze
{
  "text": "<a href=\"https://www.homegate.ch\">Hello, wörld! :)</a>",
  "explain": true
}
```

Index a document:

```
POST myindex/_doc
{
  "text": "<a href=\"https://www.homegate.ch\">Hello, wörld! :)</a>"
}

```

Check what tokens were actually stored in the document after the text analysis:

```
GET myindex/_termvectors/VY3Zw4gBjq6g8vG5Z5Ww?fields=text
```

## Text query example

```
DELETE myindex

PUT myindex
{
  "settings": {
    "number_of_replicas": 0
  },
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "kw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

POST myindex/_doc
{
  "title": "Freshly renovated, comfortable furnished apartment in quiet and beautiful neighbourhood"
}
POST myindex/_doc
{
  "title": "Recently renovated, quiet apartment with a lake view"
}

GET myindex/_termvectors/Wo1cxIgBjq6g8vG5qJV-?fields=title,title.kw

GET myindex/_search
{
  "query": {
    "match": {
      "title": "renovated"
    }
  }
}

GET myindex/_search
{
  "query": {
    "match": {
      "title": "renovated apartment"
    }
  }
}

GET myindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "quiet lake",
        "operator": "or"
      }
    }
  }
}

GET myindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "quieter",
        "fuzziness": 2
      }
    }
  }
}

GET myindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "quiet lake renovated",
        "minimum_should_match": 3
      }
    }
  }
}

```
