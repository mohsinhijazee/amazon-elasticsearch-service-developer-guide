# Supported Elasticsearch Operations<a name="aes-supported-es-operations"></a>

Amazon ES supports many versions of Elasticsearch\. The following topics show the operations that Amazon ES supports for each version\.

**Topics**
+ [Notable API Differences](#es_version_api_notes)
+ [Version 7\.1](#es_version_7_1)
+ [Version 6\.8](#es_version_6_8)
+ [Version 6\.7](#es_version_6_7)
+ [Version 6\.5](#es_version_6_5)
+ [Version 6\.4](#es_version_6_4)
+ [Version 6\.3](#es_version_6_3)
+ [Version 6\.2](#es_version_6_2)
+ [Version 6\.0](#es_version_6_0)
+ [Version 5\.6](#es_version_5_6)
+ [Version 5\.5](#es_version_5_5)
+ [Version 5\.3](#es_version_5_3)
+ [Version 5\.1](#es_version_5_1)
+ [Version 2\.3](#es_version_2_3)
+ [Version 1\.5](#es_version_1_5)

## Notable API Differences<a name="es_version_api_notes"></a>

### Cluster Settings<a name="es_version_api_notes-cs"></a>

Prior to Elasticsearch 5\.3, the `_cluster/settings` API on Amazon ES domains supported only the HTTP `PUT` method, not the `GET` method\. Newer versions support the `GET` method, as shown in the following example:

```
GET https://domain.region.es.amazonaws.com/_cluster/settings?pretty
```

A sample return follows:

```
{
  "persistent" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "cluster_concurrent_rebalance" : "2"
        }
      }
    },
    "indices" : {
      "recovery" : {
        "max_bytes_per_sec" : "20mb"
      }
    }
  },
  "transient" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "exclude" : {
            "di_number" : "2"
          }
        }
      }
    }
  }
}
```
+ `cluster_concurrent_rebalance` specifies the number of shards that can be relocated to new nodes at any given time\.
+ `max_bytes_per_sec` is the maximum data transfer speed that Elasticsearch uses during a recovery event\.
+ `di_number` is an internal Amazon ES value that is used to copy shards to new *domain instances* after configuration changes\.

### Shrink<a name="es_version_api_notes-shrink"></a>

The `_shrink` API can cause upgrades, configuration changes, and domain deletions to fail\. We don't recommend using it on domains that run Elasticsearch versions 5\.3 or 5\.1\. These versions have a bug that can cause snapshot restoration of shrunken indices to fail\.

If you use the `_shrink` API on other Elasticsearch versions, make the following request before starting the shrink operation:

```
PUT https://domain.region.es.amazonaws.com/source-index/_settings
{
  "settings": {
    "index.routing.allocation.require._name": "name-of-the-node-to-shrink-to",
    "index.blocks.read_only": true
  }
}
```

Then make the following request after completing the shrink operation:

```
PUT https://domain.region.es.amazonaws.com/source-index/_settings
{
  "settings": {
    "index.routing.allocation.require._name": null,
    "index.blocks.read_only": false
  }
}
```

## Version 7\.1<a name="es_version_7_1"></a>

For Elasticsearch 7\.1, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 6\.8<a name="es_version_6_8"></a>

For Elasticsearch 6\.8, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 6\.7<a name="es_version_6_7"></a>

For Elasticsearch 6\.7, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 6\.5<a name="es_version_6_5"></a>

For Elasticsearch 6\.5, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 6\.4<a name="es_version_6_4"></a>

For Elasticsearch 6\.4, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 6\.3<a name="es_version_6_3"></a>

For Elasticsearch 6\.3, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 6\.2<a name="es_version_6_2"></a>

For Elasticsearch 6\.2, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 6\.0<a name="es_version_6_0"></a>

For Elasticsearch 6\.0, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 5\.6<a name="es_version_5_6"></a>

For Elasticsearch 5\.6, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 5\.5<a name="es_version_5_5"></a>

For Elasticsearch 5\.5, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 5\.3<a name="es_version_5_3"></a>

For Elasticsearch 5\.3, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. Refers to the `PUT` method\. For information about the `GET` method, see [Notable API Differences](#es_version_api_notes)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 5\.1<a name="es_version_5_1"></a>

For Elasticsearch 5\.1, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html) [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

1. Cluster configuration changes might interrupt these operations before completion\. We recommend that you use the `/_tasks` operation along with these operations to verify that the requests completed successfully\.

1. DELETE requests to `/_search/scroll` with a message body must specify `"Content-Length"` in the HTTP header\. Most clients add this header by default\. To avoid a problem with `=` characters in `scroll_id` values, use the request body, not the query string, to pass `scroll_id` values to Amazon ES\.

1. For considerations about using scripts, see [Other Supported Resources](aes-supported-resources.md)\.

1. See [Shrink](#es_version_api_notes-shrink)\.

## Version 2\.3<a name="es_version_2_3"></a>

For Elasticsearch 2\.3, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 

## Version 1\.5<a name="es_version_1_5"></a>

For Elasticsearch 1\.5, Amazon ES supports the following operations\.


|  |  |  | 
| --- |--- |--- |
|  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-es-operations.html)  | 