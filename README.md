# Elasticsearch-Autocomplete
Elasticsearch Autocomplete, Autosuggestion, Type Ahead or however you want to call it!

# Welcome to the Elasticsearch-Autocomplete wiki!

Elasticsearch has several ways to implement autocomplete. One of the easiest way is to use the "match phrase prefix" query type, which is less expensive. However, Elasticsearch document refers this approach as "poor-man’s autocomplete", for obvious reasons. 

[https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase-prefix.html](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase-prefix.html)

How about giving a hand-out to the poor man? Come, let's do that.

Data is wealth. Rich data is wealthy data. So, to create a wealthy autocomplete dictionary we need rich data, data that is relevant to the search index. Here, I list down the steps to extract relevant data to generate a autocomplete dictionary.

1. Create a "temporary" Index (I have a Elastic index that is build by indexing a bunch of PDF, Word Doc, Excel and other binary fiels). The "temporary" Index will be a copy of my Original Index, expect for enabling filterdata on it. Generally, we do not enable filterdata on a Content attribute unless needed. But if you already have your Content attribute filterdata enabled then you don't need a "temporary" Index. I'm going to call the "temporary" index "terms_extract" 

[https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html)

``` PUT /terms_extract?pretty=true ```

2. Create a mapping for the "terms_extract" with just a "Content" and "id" attribute (Unless you want to extract from few other attributes)

```
PUT /terms_extract/_mapping/doc_records?pretty=true
{
  "doc_records": {
    "properties": {
      "DocContent": {
        "type": "text",
        "fielddata": true
      },
      "id": {
        "type": "text"
      }
    }
  }
}
```

3. Index your data on-to your "terms_extract" index. Just one time. And refresh/incremental the index as needed

Refer to my AWS S3 bucket to Elastic connector code if you have AWS S3 buckets as your data source. If not, do this steps based on your scenario

[https://github.com/aswath86/AWS-lambda-S3-to-Elastic-Indexing-Connector](https://github.com/aswath86/AWS-lambda-S3-to-Elastic-Indexing-Connector)

4. This is the best part. We extract the popular terms from the index. The autocomplete suggestion has to be relevant to the data present in the search index. Where else is the best place to lookup for the relevant and popular words for your data than your search index.

In this example, I'm pulling 1000 terms that are minimum 4 characters long, excluding some stop words.

You can do a whole lot of restriction and rules on this. Refer [https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html)

```
POST /terms_extract/_search?pretty=true&size=0
{
  "aggs": {
    "genres": {
      "terms": {
        "field": "DocContent",
        "include": "[a-z|0-9][a-z|0-9][a-z|0-9][a-z|0-9].*",
        "exclude" : ["they", "those", "them"],
        "size": 1000
      }
    }
  }
}
```

And, of-course, here is a good list of English stop words if you want to use,

```
[ "a", "about", "above", "after", "again", "against", "all", "am", "an", "and", "any", "are", "as", "at", "be", "because", "been", "before", "being", "below", "between", "both", "but", "by", "could", "did", "do", "does", "doing", "down", "during", "each", "few", "for", "from", "further", "had", "has", "have", "having", "he", "he'd", "he'll", "he's", "her", "here", "here's", "hers", "herself", "him", "himself", "his", "how", "how's", "i", "i'd", "i'll", "i'm", "i've", "if", "in", "into", "is", "it", "it's", "its", "itself", "let's", "me", "more", "most", "my", "myself", "nor", "of", "on", "once", "only", "or", "other", "ought", "our", "ours", "ourselves", "out", "over", "own", "same", "she", "she'd", "she'll", "she's", "should", "so", "some", "such", "than", "that", "that's", "the", "their", "theirs", "them", "themselves", "then", "there", "there's", "these", "they", "they'd", "they'll", "they're", "they've", "this", "those", "through", "to", "too", "under", "until", "up", "very", "was", "we", "we'd", "we'll", "we're", "we've", "were", "what", "what's", "when", "when's", "where", "where's", "which", "while", "who", "who's", "whom", "why", "why's", "with", "would", "you", "you'd", "you'll", "you're", "you've", "your", "yours", "yourself", "yourselves" ]
```


5. And here is the actual part where we create the "type_ahead" index that will be used for the autocomplete feature.

```
PUT /type_ahead?pretty=true
```

```
PUT /type_ahead/_mapping/doc_records?pretty=true
{
  "doc_records": {
    "properties": {
      "dictionary": {
        "type": "text",
        "fielddata": true
      },
      "count": {
        "type": "integer"
      }
    }
  }
}
```

6. and load the extracted terms to the "type_ahead" index. Let's use the Bulk API to do this. I'm only adding 10 records for this example

[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

```
POST /_bulk
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"1"}}
{"dictionary":"circuit","count":"12"}
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"2"}}
{"dictionary":"brakes","count":"11"}
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"3"}}
{"dictionary":"motor","count":"11"}
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"4"}}
{"dictionary":"engine","count":"11"}
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"5"}}
{"dictionary":"model","count":"11"}
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"6"}}
{"dictionary":"diagnostic","count":"11"}
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"7"}}
{"dictionary":"date","count":"10"}
{"create":{"_index":"fire","_type":"doc_records","_id":"8"}}
{"dictionary":"first","count":"10"}
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"9"}}
{"dictionary":"model","count":"10"}
{"create":{"_index":"type_ahead","_type":"doc_records","_id":"10"}}
{"dictionary":"moderator","count":"10"}
```


7. Finally, the purpose of this, getting the autocomplete suggestion. See how we sort it based on the count? That determines that the most popular words are suggested first. The "query" takes in the characters that are keyed-in

```
GET type_ahead/_search
{
  "query": {
    "match_phrase_prefix": {
      "dictionary": {
        "query": "mo",
        "max_expansions": 5
      }
    }
  },
  "sort": [
    {
      "count": {
        "order": "desc",
        "mode": "avg"
      }
    }
  ]
}
```

8. Well, I have used "finally" at the penultimate step so I cannot use it again, so,

### As always, don't forget to improvise!
