# Managing Amazon Elasticsearch Service Domains<a name="es-managedomains"></a>

As the size and number of documents in your Amazon Elasticsearch Service \(Amazon ES\) domain grow and as network traffic increases, you likely will need to update the configuration of your Elasticsearch cluster\. To know when it's time to reconfigure your domain, you need to monitor domain metrics\. You might also need to audit data\-related API calls to your domain or assign tags to your domain\. This section describes how to perform these and other tasks related to managing your domains\.

**Topics**
+ [About Configuration Changes](#es-managedomains-configuration-changes)
+ [Charges for Configuration Changes](#es-managedomains-config-charges)
+ [Service Software Updates](#es-service-software)
+ [Configuring a Multi\-AZ Domain](#es-managedomains-multiaz)
+ [Interpreting Health Dashboards](#es-managedomains-cloudwatchmetrics-box-charts)
+ [Monitoring Cluster Metrics and Statistics with Amazon CloudWatch \(Console\)](#es-managedomains-cloudwatchmetrics)
+ [Tagging Amazon Elasticsearch Service Domains](#es-managedomains-awsresourcetagging)

## About Configuration Changes<a name="es-managedomains-configuration-changes"></a>

Amazon ES uses a *blue/green* deployment process when updating domains\. Blue/green typically refers to the practice of running two production environments, one live and one idle, and switching the two as you make software changes\. In the case of Amazon ES, it refers to the practice of creating a new environment for domain updates and routing users to the new environment after those updates are complete\. The practice minimizes downtime and maintains the original environment in the event that deployment to the new environment is unsuccessful\.

The following operations cause blue/green deployments:
+ Changing instance type
+ If your domain *doesn't* have dedicated master nodes, changing data instance count
+ Enabling or disabling dedicated master nodes
+ Changing dedicated master node count
+ Enabling or disabling Multi\-AZ
+ Changing storage type, volume type, or volume size
+ Choosing different VPC subnets
+ Adding or removing VPC security groups
+ Enabling or disabling Amazon Cognito authentication for Kibana
+ Choosing a different Amazon Cognito user pool or identity pool
+ Modifying advanced settings
+ Enabling or disabling the publication of error logs or slow logs to CloudWatch
+ Upgrading to a new Elasticsearch version

In *most* cases, the following operations do not cause blue/green deployments:
+ Changing access policy
+ Changing the automated snapshot hour
+ If your domain has dedicated master nodes, changing data instance count

Exceptions exist\. For example, if you have not reconfigured your domain since the launch of three Availability Zone support, Amazon ES might perform a one\-time blue/green deployment to redistribute your dedicated master nodes across Availability Zones\.

If you initiate a configuration change, the domain state changes to **Processing**\. During certain [service software updates](#es-service-software), the state remains **Active**\. In both cases, you can review the cluster health and Amazon CloudWatch metrics and see that the number of nodes in the cluster temporarily increases—often doubling—while the domain update occurs\. In the following illustration, you can see the number of nodes doubling from 11 to 22 during a configuration change and returning to 11 when the update is complete\.

![\[Number of nodes doubling from 11 to 22 during a domain configuration change.\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/NodesDoubled.png)

This temporary increase can strain the cluster's [dedicated master nodes](es-managedomains-dedicatedmasternodes.md), which suddenly have many more nodes to manage\. It is important to maintain sufficient capacity on dedicated master nodes to handle the overhead that is associated with these blue/green deployments\.

**Important**  
You do *not* incur any additional charges during configuration changes and service maintenance\. You are billed only for the number of nodes that you request for your cluster\. For specifics, see [Charges for Configuration Changes](#es-managedomains-config-charges)\.

To prevent overloading dedicated master nodes, you can [monitor usage with the Amazon CloudWatch metrics](#es-managedomains-cloudwatchmetrics)\. For recommended maximum values, see [Recommended CloudWatch Alarms](cloudwatch-alarms.md)\.

## Charges for Configuration Changes<a name="es-managedomains-config-charges"></a>

If you change the configuration for a domain, Amazon ES creates a new cluster as described in [About Configuration Changes](#es-managedomains-configuration-changes)\. During the migration of old to new, you incur the following charges:
+ If you change the instance type, you are charged for both clusters for the first hour\. After the first hour, you are charged only for the new cluster\.

  **Example:** You change the configuration from three `m3.xlarge` instances to four `m4.large` instances\. For the first hour, you are charged for both clusters \(3 \* `m3.xlarge` \+ 4 \* `m4.large`\)\. After the first hour, you are charged only for the new cluster \(4 \* `m4.large`\)\.
+ If you don’t change the instance type, you are charged only for the largest cluster for the first hour\. After the first hour, you are charged only for the new cluster\.

  **Example:** You change the configuration from six `m3.xlarge` instances to three `m3.xlarge` instances\. For the first hour, you are charged for the largest cluster \(6 \* `m3.xlarge`\)\. After the first hour, you are charged only for the new cluster \(3 \* `m3.xlarge`\)\.

## Service Software Updates<a name="es-service-software"></a>

**Note**  
Service software updates differ from Elasticsearch version upgrades\. For information about upgrading to a later version of Elasticsearch, see [Upgrading Elasticsearch](es-version-migration.md)\.

Amazon ES regularly releases system software updates that add features or otherwise improve your domains\. The console is the easiest way to see if an update is available\. For a history of service software updates, see [Release Notes](release-notes.md#release-table)\.

When new service software becomes available, you can request an update to your domain and benefit from new features immediately\. You might also want to start the update at a low traffic time\.

Some updates are required, and others are optional\. If you take no action on required updates, we still update the service software automatically after a certain timeframe \(typically two weeks\)\. Your domain might be ineligible for a service software update if it is in any of the states that are shown in the following table\.


| State | Description | 
| --- | --- | 
| Domain in processing | The domain is in the middle of a configuration change\. Check update eligibility after the operation completes\. | 
| Red cluster status | One or more indices in the cluster is red\. For troubleshooting steps, see [Red Cluster Status](aes-handling-errors.md#aes-handling-errors-red-cluster-status)\. | 
| High error rate | The Elasticsearch cluster is returning a large number of 5xx errors when attempting to process requests\. This problem is usually the result of too many simultaneous read or write requests\. Consider reducing traffic to the cluster or scaling your domain\. | 
| Split brain | Split brain means that your Elasticsearch cluster has more than one master node and has split into two clusters that never will rejoin on their own\. You can avoid split brain by using the recommended number of [dedicated master nodes](es-managedomains-dedicatedmasternodes.md)\. For help recovering from split brain, contact [AWS Support](https://console.aws.amazon.com/support/home)\. | 
| Amazon Cognito integration issue | Your domain uses [authentication for Kibana](es-cognito-auth.md), and Amazon ES can't find one or more Amazon Cognito resources\. This problem usually occurs if the Amazon Cognito user pool is missing\. To correct the issue, recreate the missing resource and configure the Amazon ES domain to use it\. | 
| Other Amazon ES service issue | Issues with Amazon ES itself might cause your domain to display as ineligible for an update\. If none of the previous conditions apply to your domain and the problem persists for more than a day, contact [AWS Support](https://console.aws.amazon.com/support/home)\. | 

**To request a service software update \(console\)**

1. Go to [https://aws\.amazon\.com](https://aws.amazon.com), and then choose **Sign In to the Console**\.

1. Under **Analytics**, choose **Elasticsearch Service**\.

1. In the navigation pane, under **My domains**, choose the domain that you want to update\.

1. For **Service software release**, use the documentation link to compare your current version to the latest version\. Then choose **Update**\.

**To request a service software update \(AWS CLI and AWS SDKs\)**

You can use the following commands to see if an update is available, check upgrade eligibility, and request an update:
+ `describe-elasticsearch-domain` \(`DescribeElasticsearchDomain`\)
+ `start-elasticsearch-service-software-update` \(`StartElasticsearchServiceSoftwareUpdate`\)

For more information, see the [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/) and [Amazon Elasticsearch Service Configuration API Reference](es-configuration-api.md)\.

**Tip**  
After requesting an update, you might have a narrow window of time in which you can cancel it\. Use the console or `stop-elasticsearch-service-software-update` \(`StopElasticsearchServiceSoftwareUpdate`\) command\.

## Configuring a Multi\-AZ Domain<a name="es-managedomains-multiaz"></a>

Each AWS Region is a separate geographic area with multiple, isolated locations known as *Availability Zones*\. To prevent data loss and minimize cluster downtime in the event of a service disruption, you can distribute nodes across two or three Availability Zones in the same Region, a configuration known as *Multi\-AZ*\.

For domains that run production workloads, we recommend the following configuration:
+ Choose a Region that supports three Availability Zones with Amazon ES:
  + US East \(N\. Virginia, Ohio\)
  + US West \(Oregon\)
  + EU \(Frankfurt, Ireland, London, Paris\)
  + Asia Pacific \(Singapore, Sydney, Tokyo\)
  + China \(Ningxia\)
+ Deploy the domain across three zones\.
+ Choose current\-generation instance types for dedicated master nodes and data nodes\.
+ Use three dedicated master nodes and at least three data nodes\.
+ Create at least one replica for each index in your cluster\.

The rest of this section provides explanations for and context around these recommendations\.

### Shard Distribution<a name="es-managedomains-za-shards"></a>

If you enable Multi\-AZ, you should create at least one replica for each index in your cluster\. Without replicas, Amazon ES can't distribute copies of your data to other Availability Zones, which largely defeats the purpose of Multi\-AZ\. Fortunately, the default configuration for any index is a replica count of 1\. As the following diagram shows, Amazon ES distributes primary and replica shards so that no replica shard is in the same zone as its corresponding primary shard\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/za-3-az.png)

In addition to distributing shards by Availability Zone, Amazon ES distributes them by node\. Still, certain domain configurations can result in imbalanced shard counts\. Consider the following domain:
+ 5 data nodes
+ 5 primary shards
+ 2 replicas
+ 3 Availability Zones

In this situation, Amazon ES has to overload one node in order to distribute the primary and replica shards across the zones, as shown in the following diagram\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/za-3-az-imbal.png)

To avoid these kinds of situations, which can strain individual nodes and hurt performance, we recommend that you choose an instance count that is a multiple of three if you plan to have two or more replicas per index\.

### Dedicated Master Node Distribution<a name="es-managedomains-za-dm"></a>

Even if you select two Availability Zones when configuring your domain, Amazon ES automatically distributes [dedicated master nodes](es-managedomains-dedicatedmasternodes.md) across three Availability Zones\. This distribution helps prevent cluster downtime if a zone experiences a service disruption\. If you use the recommended three dedicated master nodes and one Availability Zone goes down, your cluster still has a quorum \(2\) of dedicated master nodes and can elect a new master\. The following diagram demonstrates this configuration\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/za-2-az.png)

This automatic distribution has some notable exceptions:
+ If you choose an older\-generation instance type that is not available in three Availability Zones, the following scenarios apply:
  + If you chose three Availability Zones for the domain, Amazon ES throws an error\. Choose a different instance type, and try again\.
  + If you chose two Availability Zones for the domain, Amazon ES distributes the dedicated master nodes across two zones\.
+ Not all AWS Regions have three Availability Zones\. In these Regions, you can only configure a domain to use two zones \(and Amazon ES can only distribute dedicated master nodes across two zones\)\. For the list of Regions that support three Availability Zones, see [Configuring a Multi\-AZ Domain](#es-managedomains-multiaz)\.

### Availability Zone Disruptions<a name="es-managedomains-za-summary"></a>

Availability Zone disruptions are rare, but do occur\. The following table lists different Multi\-AZ configurations and behaviors during a disruption\.


| Number of Availability Zones in a Region | Number of Availability Zones That You Chose | Number of Dedicated Master Nodes | Behavior if One Availability Zone Experiences a Disruption | 
| --- | --- | --- | --- | 
| 2 or more | 2 | 0 |  Downtime\. Your cluster loses half of its data nodes and must replace at least one in the remaining Availability Zone before it can elect a master\.  | 
| 2 | 2 | 3 |  50/50 chance of downtime\. Amazon ES distributes two dedicated master nodes into one Availability Zone and one into the other: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-managedomains.html)  | 
| 3 or more | 2 | 3 |  No downtime\. Amazon ES automatically distributes the dedicated master nodes across three Availability Zones, so the remaining two dedicated master nodes can elect a master\.  | 
| 3 or more | 3 | 0 |  No downtime\. Roughly two\-thirds of your data nodes are still available to elect a master\.  | 
| 3 or more | 3 | 3 |  No downtime\. The remaining two dedicated master nodes can elect a master\.  | 

In *all* configurations, regardless of the cause, node failures can cause the cluster's remaining data nodes to experience a period of increased load while Amazon ES automatically configures new nodes to replace the now\-missing ones\.

For example, in the event of an Availability Zone disruption in a three\-zone configuration, two\-thirds as many data nodes have to process just as many requests to the cluster\. As they process these requests, the remaining nodes are also replicating shards onto new nodes as they come online, which can further impact performance\. If availability is critical to your workload, consider adding resources to your cluster to alleviate this concern\.

## Interpreting Health Dashboards<a name="es-managedomains-cloudwatchmetrics-box-charts"></a>

The **Instance health** tab in the Amazon ES console uses box charts to provide at\-a\-glance visibility into the health of each Elasticsearch node\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/box-charts.png)
+ Each colored box shows the range of values for the node over the specified time period\.
+ Blue boxes represent values that are consistent with other nodes\. Red boxes represent outliers\.
+ The white line within each box shows the node's current value\.
+ The “whiskers” on either side of each box show the minimum and maximum values for all nodes over the time period\.

## Monitoring Cluster Metrics and Statistics with Amazon CloudWatch \(Console\)<a name="es-managedomains-cloudwatchmetrics"></a>

Amazon ES domains send performance metrics to Amazon CloudWatch every minute\. If you use General Purpose or Magnetic EBS volumes, the EBS volume metrics update only every five minutes\. To view these metrics, use the **Cluster health** and **Instance health** tabs in the Amazon Elasticsearch Service console\. The metrics are provided at no extra charge\.

If you make configuration changes to your domain, the list of individual instances in the **Cluster health** and **Instance health** tabs often double in size for a brief period before returning to the correct number\. For an explanation of this behavior, see [About Configuration Changes](#es-managedomains-configuration-changes)\.

Amazon ES metrics fall into these categories:
+ [Cluster Metrics](#es-managedomains-cloudwatchmetrics-cluster-metrics)
+ [Dedicated Master Node Metrics](#es-managedomains-cloudwatchmetrics-master-node-metrics)
+ [EBS Volume Metrics](#es-managedomains-cloudwatchmetrics-master-ebs-metrics)
+ [Instance Metrics](#es-managedomains-cloudwatchmetrics-instance-metrics)

**Note**  
The service archives the metrics for two weeks before discarding them\.

### Cluster Metrics<a name="es-managedomains-cloudwatchmetrics-cluster-metrics"></a>

The `AWS/ES` namespace includes the following metrics for clusters\.


| Metric | Description | 
| --- | --- | 
| ClusterStatus\.green | Indicates that all index shards are allocated to nodes in the cluster\. Relevant statistics: Minimum, Maximum | 
| ClusterStatus\.yellow | Indicates that the primary shards for all indices are allocated to nodes in a cluster, but the replica shards for at least one index are not\. Single node clusters always initialize with this cluster status because there is no second node to which a replica can be assigned\. You can either increase your node count to obtain a green cluster status, or you can use the Elasticsearch API to set the number\_of\_replicas setting for your index to 0\. To learn more, see [Configuring Amazon Elasticsearch Service Domains](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-createupdatedomains.html#es-createdomains-configure-cluster)\.Relevant statistics: Minimum, Maximum | 
| ClusterStatus\.red | Indicates that the primary and replica shards of at least one index are not allocated to nodes in a cluster\. To recover, you must delete the indices or restore a snapshot and then add EBS\-based storage, use larger instance types, or add instances\. For more information, see [Red Cluster Status](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-handling-errors.html#aes-handling-errors-red-cluster-status)\. Relevant statistics: Minimum, Maximum | 
| Nodes | The number of nodes in the Amazon ES cluster, including dedicated master nodes\. Relevant statistics: Minimum, Maximum, Average | 
| SearchableDocuments | The total number of searchable documents across all indices in the cluster\. Relevant statistics: Minimum, Maximum, Average | 
| DeletedDocuments | The total number of documents marked for deletion across all indices in the cluster\. These documents no longer appear in search results, but Elasticsearch only removes deleted documents from disk during segment merges\. This metric increases after delete requests and decreases after segment merges\. Relevant statistics: Minimum, Maximum, Average | 
| CPUUtilization | The maximum percentage of CPU resources used for data nodes in the cluster\. Relevant statistics: Maximum, Average | 
| FreeStorageSpace | The free space, in GiB, for nodes in the cluster\. `Sum` shows total free space for the cluster, but you must leave the period at one minute to get an accurate value\. `Minimum`, `Maximum`, and `Average` show free space for individual nodes\. Amazon ES throws a `ClusterBlockException` when this metric reaches `0`\. To recover, you must either delete indices, add larger instances, or add EBS\-based storage to existing instances\. To learn more, see [Recovering from a Lack of Free Storage Space](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-handling-errors.html#aes-handling-errors-watermark) `FreeStorageSpace` will always be lower than the value that the Elasticsearch `_cluster/stats` API provides\. Amazon ES reserves a percentage of the storage space on each instance for internal operations\. Relevant statistics: Minimum, Maximum, Average, Sum | 
| ClusterUsedSpace | The total used space, in GiB, for the cluster\. You can view this metric in the Amazon CloudWatch console, but not in the Amazon ES console\. You must leave the period at one minute to get an accurate value\. Relevant statistics: Minimum, Maximum | 
| ClusterIndexWritesBlocked | Indicates whether your cluster is accepting or blocking incoming write requests\. A value of 0 means that the cluster is accepting requests\. A value of 1 means that it is blocking requests\. Many factors can cause a cluster to begin blocking requests\. Some common factors include the following: `FreeStorageSpace` is too low, `JVMMemoryPressure` is too high, or `CPUUtilization` is too high\. To alleviate this issue, consider adding more disk space or scaling your cluster\. Relevant statistics: Maximum You can view this metric in the Amazon CloudWatch console, but not the Amazon ES console\. | 
| JVMMemoryPressure | The maximum percentage of the Java heap used for all data nodes in the cluster\. Relevant statistics: Maximum | 
| AutomatedSnapshotFailure | The number of failed automated snapshots for the cluster\. A value of `1` indicates that no automated snapshot was taken for the domain in the previous 36 hours\. Relevant statistics: Minimum, Maximum | 
| CPUCreditBalance | The remaining CPU credits available for data nodes in the cluster\. A CPU credit provides the performance of a full CPU core for one minute\. For more information, see [CPU Credits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/t2-instances.html#t2-instances-cpu-credits) in the *Amazon EC2 Developer Guide*\. This metric is available only for the t2\.micro\.elasticsearch, t2\.small\.elasticsearch, and t2\.medium\.elasticsearch instance types\. Relevant statistics: Minimum | 
| KibanaHealthyNodes | A health check for Kibana\. A value of 1 indicates normal behavior\. A value of 0 indicates that Kibana is inaccessible\. In most cases, the health of Kibana mirrors the health of the cluster\. Relevant statistics: Minimum You can view this metric on the Amazon CloudWatch console, but not the Amazon ES console\. | 
| KMSKeyError | A value of 1 indicates that the KMS customer master key used to encrypt data at rest has been disabled\. To restore the domain to normal operations, re\-enable the key\. The console displays this metric only for domains that encrypt data at rest\. Relevant statistics: Minimum, Maximum | 
| KMSKeyInaccessible | A value of 1 indicates that the KMS customer master key used to encrypt data at rest has been deleted or revoked its grants to Amazon ES\. You can't recover domains that are in this state\. If you have a manual snapshot, though, you can use it to migrate the domain's data to a new domain\. The console displays this metric only for domains that encrypt data at rest\. Relevant statistics: Minimum, Maximum | 
| InvalidHostHeaderRequests | The number of HTTP requests made to the Elasticsearch cluster that included an invalid \(or missing\) host header\. Valid requests include the domain hostname as the host header value\. Amazon ES rejects invaild requests for public access domains that don't have a restrictive access policy\. We recommend applying a restrictive access policy to all domains\.If you see large values for this metric, confirm that your Elasticsearch clients include the domain hostname \(and not, for example, its IP address\) in their requests\. Relevant statistics: Sum | 
| ElasticsearchRequests | The number of requests made to the Elasticsearch cluster\. Relevant statistics: Sum | 
| RequestCount | The number of requests to a domain and the HTTP response code \(2xx, 3xx, 4xx, 5xx\) for each request\. Relevant statistics: Sum | 

### Dedicated Master Node Metrics<a name="es-managedomains-cloudwatchmetrics-master-node-metrics"></a>

The `AWS/ES` namespace includes the following metrics for dedicated master nodes\.


| Metric | Description | 
| --- | --- | 
| MasterCPUUtilization | The maximum percentage of CPU resources used by the dedicated master nodes\. We recommend increasing the size of the instance type when this metric reaches 60 percent\. Relevant statistics: Average | 
| MasterFreeStorageSpace | This metric is not relevant and can be ignored\. The service does not use master nodes as data nodes\. | 
| MasterJVMMemoryPressure | The maximum percentage of the Java heap used for all dedicated master nodes in the cluster\. We recommend moving to a larger instance type when this metric reaches 85 percent\. Relevant statistics: Maximum | 
| MasterCPUCreditBalance | The remaining CPU credits available for dedicated master nodes in the cluster\. A CPU credit provides the performance of a full CPU core for one minute\. For more information, see [CPU Credits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/t2-instances.html#t2-instances-cpu-credits) in the *Amazon EC2 User Guide for Linux Instances*\. This metric is available only for the t2\.micro\.elasticsearch, t2\.small\.elasticsearch, and t2\.medium\.elasticsearch instance types\. Relevant statistics: Minimum | 
| MasterReachableFromNode | A health check for `MasterNotDiscovered` exceptions\. A value of 1 indicates normal behavior\. A value of 0 indicates that `/_cluster/health/` is failing\. Failures mean that the master node stopped or is not reachable\. They are usually the result of a network connectivity issue or AWS dependency problem\. Relevant statistics: Minimum You can view this metric on the Amazon CloudWatch console, but not the Amazon ES console\. | 
| MasterSysMemoryUtilization | The percentage of the master node's memory that is in use\. Relevant statistics: Maximum | 

### EBS Volume Metrics<a name="es-managedomains-cloudwatchmetrics-master-ebs-metrics"></a>

The `AWS/ES` namespace includes the following metrics for EBS volumes\.


| Metric | Description | 
| --- | --- | 
| ReadLatency | The latency, in seconds, for read operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average | 
| WriteLatency | The latency, in seconds, for write operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average | 
| ReadThroughput | The throughput, in bytes per second, for read operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average | 
| WriteThroughput | The throughput, in bytes per second, for write operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average | 
| DiskQueueDepth | The number of pending input and output \(I/O\) requests for an EBS volume\. Relevant statistics: Minimum, Maximum, Average | 
| ReadIOPS | The number of input and output \(I/O\) operations per second for read operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average | 
| WriteIOPS | The number of input and output \(I/O\) operations per second for write operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average | 

### Instance Metrics<a name="es-managedomains-cloudwatchmetrics-instance-metrics"></a>

The `AWS/ES` namespace includes the following metrics for each instance in a domain\. Amazon ES also aggregates these instance metrics to provide insight into overall cluster health\. You can verify this behavior using the **Data samples** statistic in the console\. Note that each metric in the following table has relevant statistics for the node *and* the cluster\.

**Important**  
Different versions of Elasticsearch use different thread pools to process calls to the `_index` API\. Elasticsearch 1\.5 and 2\.3 use the index thread pool\. Elasticsearch 5\.*x*, 6\.0, and 6\.2 use the bulk thread pool\. 6\.3 and newer use the write thread pool\. Currently, Amazon ES doesn't provide metrics for the bulk and write thread pools\.


| Metric | Description | 
| --- | --- | 
| IndexingLatency | The average time, in milliseconds, that it takes a shard to complete an indexing operation\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum | 
| IndexingRate | The number of indexing operations per minute\. A single call to the `_bulk` API that adds two documents and updates two counts as four operations, which might be spread across one or more nodes\. If that index has one or more replicas, other nodes in the cluster also record a total of four indexing operations\. Document deletions do not count towards this metric\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum, Sum | 
| SearchLatency | The average time, in milliseconds, that it takes a shard to complete a search operation\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum | 
| SearchRate | The total number of search requests per minute for all shards on a node\. A single call to the `_search` API might return results from many different shards\. If five of these shards are on one node, the node would report 5 for this metric, even though the client only made one request\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum, Sum | 
| SysMemoryUtilization | The percentage of the instance's memory that is in use\. Relevant node statistics: Minimum, Maximum, Average Relevant cluster statistics: Minimum, Maximum, Average | 
| JVMGCYoungCollectionCount | The number of times that "young generation" garbage collection has run\. A large, ever\-growing number of runs is a normal part of cluster operations\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average | 
| JVMGCYoungCollectionTime | The amount of time, in milliseconds, that the cluster has spent performing "young generation" garbage collection\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average | 
| JVMGCOldCollectionCount | The number of times that "old generation" garbage collection has run\. In a cluster with sufficient resources, this number should remain small and grow infrequently\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average | 
| JVMGCOldCollectionTime | The amount of time, in milliseconds, that the cluster has spent performing "old generation" garbage collection\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average | 
| ThreadpoolForce\_mergeQueue | The number of queued tasks in the force merge thread pool\. If the queue size is consistently high, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average | 
| ThreadpoolForce\_mergeRejected | The number of rejected tasks in the force merge thread pool\. If this number continually grows, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum | 
| ThreadpoolForce\_mergeThreads | The size of the force merge thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum | 
| ThreadpoolIndexQueue | The number of queued tasks in the index thread pool\. If the queue size is consistently high, consider scaling your cluster\. The maximum index queue size is 200\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average | 
| ThreadpoolIndexRejected | The number of rejected tasks in the index thread pool\. If this number continually grows, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum | 
| ThreadpoolIndexThreads | The size of the index thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum | 
| ThreadpoolSearchQueue | The number of queued tasks in the search thread pool\. If the queue size is consistently high, consider scaling your cluster\. The maximum search queue size is 1,000\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average | 
| ThreadpoolSearchRejected | The number of rejected tasks in the search thread pool\. If this number continually grows, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum | 
| ThreadpoolSearchThreads | The size of the search thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum | 
| ThreadpoolBulkQueue | The number of queued tasks in the bulk thread pool\. If the queue size is consistently high, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average | 
| ThreadpoolBulkRejected | The number of rejected tasks in the bulk thread pool\. If this number continually grows, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum | 
| ThreadpoolBulkThreads | The size of the bulk thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum | 
| ThreadpoolWriteThreads | The size of the write thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum | 
| ThreadpoolWriteRejected | The number of rejected tasks in the write thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum | 
| ThreadpoolWriteQueue | The number of queued tasks in the write thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum | 

## Tagging Amazon Elasticsearch Service Domains<a name="es-managedomains-awsresourcetagging"></a>

You can use Amazon ES tags to add metadata to your Amazon ES domains\. AWS does not apply any semantic meaning to your tags\. Tags are interpreted strictly as character strings\. All tags have the following elements\.


****  

| Tag Element | Description | 
| --- | --- | 
| Tag key | The tag key is the required name of the tag\. Tag keys must be unique for the Amazon ES domain to which they are attached\. For a list of basic restrictions on tag keys and values, see [User\-Defined Tag Restrictions](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/allocation-tag-restrictions.html)\. | 
| Tag value | The tag value is an optional string value of the tag\. Tag values can be null and do not have to be unique in a tag set\. For example, you can have a key\-value pair in a tag set of project/Trinity and cost\-center/Trinity\. For a list of basic restrictions on tag keys and values, see [User\-Defined Tag Restrictions](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/allocation-tag-restrictions.html)\. | 

Each Amazon ES domain has a tag set, which contains all the tags that are assigned to that Amazon ES domain\. AWS does not automatically set any tags on Amazon ES domains\. A tag set can contain up to 50 tags, or it can be empty\. If you add a tag to an Amazon ES domain that has the same key as an existing tag for a resource, the new value overwrites the old value\. 

You can use these tags to track costs by grouping expenses for similarly tagged resources\. An Amazon ES domain tag is a name\-value pair that you define and associate with an Amazon ES domain\. The name is referred to as the *key*\. You can use tags to assign arbitrary information to an Amazon ES domain\. A tag key could be used, for example, to define a category, and the tag value could be an item in that category\. For example, you could define a tag key of “project” and a tag value of “Salix,” indicating that the Amazon ES domain is assigned to the Salix project\. You could also use tags to designate Amazon ES domains as being used for test or production by using a key such as `environment=test` or `environment=production`\. We recommend that you use a consistent set of tag keys to make it easier to track metadata that is associated with Amazon ES domains\. 

You also can use tags to organize your AWS bill to reflect your own cost structure\. To do this, sign up to get your AWS account bill with tag key values included\. Then, organize your billing information according to resources with the same tag key values to see the cost of combined resources\. For example, you can tag several Amazon ES domains with key\-value pairs, and then organize your billing information to see the total cost for each domain across several services\. For more information, see [Using Cost Allocation Tags](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management* documentation\.

**Note**  
Tags are cached for authorization purposes\. Because of this, additions and updates to tags on Amazon ES domains might take several minutes before they are available\.

### Working with Tags \(Console\)<a name="es-managedomains-awsresourcetagging-console"></a>

Use the following procedure to create a resource tag\.

**To create a tag \(console\)**

1. Go to [https://aws\.amazon\.com](https://aws.amazon.com), and then choose **Sign In to the Console**\.

1. Under **Analytics**, choose **Elasticsearch Service**\.

1. In the navigation pane, choose your Amazon ES domain\.

1. On the domain dashboard, choose **Manage tags**\.

1. In the **Key** column, enter a tag key\.

1. \(Optional\) In the **Value** column, enter a tag value\.

1. Choose **Submit**\.

**To delete a tag \(console\)**

Use the following procedure to delete a resource tag\.

1. Go to [https://aws\.amazon\.com](https://aws.amazon.com), and then choose **Sign In to the Console**\.

1. Under **Analytics**, choose **Elasticsearch Service**\.

1. In the navigation pane, choose your Amazon ES domain\.

1. On the domain dashboard, choose **Manage tags**\.

1. Next to the tag that you want to delete, choose **Remove**\.

1. Choose **Submit**\.

For more information about using the console to work with tags, see [Working with Tag Editor](https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/tag-editor.html) in the *AWS Management Console Getting Started Guide*\.

### Working with Tags \(AWS CLI\)<a name="es-managedomains-awsresourcetagging-cli"></a>

You can create resource tags using the AWS CLI with the \-\-add\-tags command\. 

**Syntax**

`add-tags --arn=<domain_arn> --tag-list Key=<key>,Value=<value>`


****  

| Parameter | Description | 
| --- | --- | 
| \-\-arn | Amazon resource name for the Amazon ES domain to which the tag is attached\. | 
| \-\-tag\-list | Set of space\-separated key\-value pairs in the following format: Key=<key>,Value=<value> | 

**Example**

The following example creates two tags for the *logs* domain:

```
aws es add-tags --arn arn:aws:es:us-east-1:379931976431:domain/logs --tag-list Key=service,Value=Elasticsearch Key=instances,Value=m3.2xlarge
```

You can remove tags from an Amazon ES domain using the remove\-tags command\. 

** Syntax **

`remove-tags --arn=<domain_arn> --tag-keys Key=<key>,Value=<value>`


****  

| Parameter | Description | 
| --- | --- | 
| \-\-arn | Amazon Resource Name \(ARN\) for the Amazon ES domain to which the tag is attached\. | 
| \-\-tag\-keys | Set of space\-separated key\-value pairs that you want to remove from the Amazon ES domain\. | 

**Example**

The following example removes two tags from the *logs* domain that were created in the preceding example:

```
aws es remove-tags --arn arn:aws:es:us-east-1:379931976431:domain/logs --tag-keys service instances
```

You can view the existing tags for an Amazon ES domain with the list\-tags command:

**Syntax**

`list-tags --arn=<domain_arn>`


****  

| Parameter | Description | 
| --- | --- | 
| \-\-arn | Amazon Resource Name \(ARN\) for the Amazon ES domain to which the tags are attached\. | 

**Example**

The following example lists all resource tags for the *logs* domain:

```
aws es list-tags --arn arn:aws:es:us-east-1:379931976431:domain/logs
```

### Working with Tags \(AWS SDKs\)<a name="es-managedomains-awsresourcetagging-sdk"></a>

The AWS SDKs \(except the Android and iOS SDKs\) support all the actions defined in the [Amazon ES Configuration API Reference](es-configuration-api.md), including the `AddTags`, `ListTags`, and `RemoveTags` operations\. For more information about installing and using the AWS SDKs, see [AWS Software Development Kits](http://aws.amazon.com/code)\. 