---
layout: default
title: Permissions
parent: Access control
nav_order: 110
redirect_from:
  - /security-plugin/access-control/permissions/
---

# Permissions

Each permission in the security plugin controls access to some action that the OpenSearch cluster can perform, such as indexing a document or checking cluster health.

Most permissions are self-describing. For example, `cluster:admin/ingest/pipeline/get` lets you retrieve information about ingest pipelines. _In many cases_, a permission correlates to a specific REST API operation, such as `GET _ingest/pipeline`.

Despite this correlation, permissions do **not** directly map to REST API operations. Operations such as `POST _bulk` and `GET _msearch` can access many indices and perform many actions in a single request. Even a simple request, such as `GET _cat/nodes`, performs several actions in order to generate its response.

In short, controlling access to the REST API is insufficient. Instead, the security plugin controls access to the underlying OpenSearch actions.

For example, consider the following `_bulk` request:

```json
POST _bulk
{ "delete": { "_index": "test-index", "_id": "tt2229499" } }
{ "index": { "_index": "test-index", "_id": "tt1979320" } }
{ "title": "Rush", "year": 2013 }
{ "create": { "_index": "test-index", "_id": "tt1392214" } }
{ "title": "Prisoners", "year": 2013 }
{ "update": { "_index": "test-index", "_id": "tt0816711" } }
{ "doc" : { "title": "World War Z" } }

```

For this request to succeed, you must have the following permissions for `test-index`:

- indices:data/write/bulk*
- indices:data/write/delete
- indices:data/write/index
- indices:data/write/update

These permissions also allow you add, update, or delete documents (e.g. `PUT test-index/_doc/tt0816711`), because they govern the underlying OpenSearch actions of indexing and deleting documents rather than a specific API path and HTTP method.


## Test permissions

If you want a user to have the absolute minimum set of permissions necessary to perform some function—the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege)—the best way is to send representative requests to your cluster as a new test user. In the case of a permissions error, the security plugin is very explicit about which permissions are missing. Consider this request and response:

```json
GET _cat/shards?v

{
  "error": {
    "root_cause": [{
      "type": "security_exception",
      "reason": "no permissions for [indices:monitor/stats] and User [name=test-user, backend_roles=[], requestedTenant=null]"
    }]
  },
  "status": 403
}
```

[Create a user and a role]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/), map the role to the user, and start sending signed requests using curl, Postman, or any other client. Then gradually add permissions to the role as you encounter errors. Even after you resolve one permissions error, the same request might generate new errors; the plugin only returns the first error it encounters, so keep trying until the request succeeds.

Rather than individual permissions, you can often achieve your desired security posture using a combination of the default action groups. See [Default action groups]({{site.url}}{{site.baseurl}}/security/access-control/default-action-groups/) for descriptions of the permissions that each group grants.
{: .tip }


## Cluster permissions

These permissions are for the cluster and can't be applied granularly. For example, you either have permissions to take snapshots (`cluster:admin/snapshot/create`) or you don't. The cluster permission, therefore, cannot grant a user privileges to take snapshots of a select set of indexes while preventing the user from taking snapshots of others.

Cross-references to API documentation in the permissions that follow are only intended to provide an understanding of the permissions. As stated at the beginning of this section, permissions often correlate to APIs but do not map directly to them.
{: .note }


### Ingest API permissions

See [Ingest APIs]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/).

- cluster:admin/ingest/pipeline/delete
- cluster:admin/ingest/pipeline/get
- cluster:admin/ingest/pipeline/put
- cluster:admin/ingest/pipeline/simulate
- cluster:admin/ingest/processor/grok/get

### Anomaly Detection permissions

See [Anomaly detection API]({{site.url}}{{site.baseurl}}/observing-your-data/ad/api/).

- cluster:admin/opendistro/ad/detector/delete
- cluster:admin/opendistro/ad/detector/info
- cluster:admin/opendistro/ad/detector/jobmanagement
- cluster:admin/opendistro/ad/detector/preview
- cluster:admin/opendistro/ad/detector/run
- cluster:admin/opendistro/ad/detector/search
- cluster:admin/opendistro/ad/detector/stats
- cluster:admin/opendistro/ad/detector/write
- cluster:admin/opendistro/ad/detector/validate
- cluster:admin/opendistro/ad/detectors/get
- cluster:admin/opendistro/ad/result/search
- cluster:admin/opendistro/ad/result/topAnomalies
- cluster:admin/opendistro/ad/tasks/search

### Alerting permissions

See [Alerting API]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/api/).

- cluster:admin/opendistro/alerting/alerts/ack
- cluster:admin/opendistro/alerting/alerts/get
- cluster:admin/opendistro/alerting/destination/delete
- cluster:admin/opendistro/alerting/destination/email_account/delete
- cluster:admin/opendistro/alerting/destination/email_account/get
- cluster:admin/opendistro/alerting/destination/email_account/search
- cluster:admin/opendistro/alerting/destination/email_account/write
- cluster:admin/opendistro/alerting/destination/email_group/delete
- cluster:admin/opendistro/alerting/destination/email_group/get
- cluster:admin/opendistro/alerting/destination/email_group/search
- cluster:admin/opendistro/alerting/destination/email_group/write
- cluster:admin/opendistro/alerting/destination/get
- cluster:admin/opendistro/alerting/destination/write
- cluster:admin/opendistro/alerting/monitor/delete
- cluster:admin/opendistro/alerting/monitor/execute
- cluster:admin/opendistro/alerting/monitor/get
- cluster:admin/opendistro/alerting/monitor/search
- cluster:admin/opendistro/alerting/monitor/write

### Asynchronous Search permissions

See [Asynchronous search]({{site.url}}{{site.baseurl}}/search-plugins/async/index/).

- cluster:admin/opendistro/asynchronous_search/stats
- cluster:admin/opendistro/asynchronous_search/delete
- cluster:admin/opendistro/asynchronous_search/get
- cluster:admin/opendistro/asynchronous_search/submit

### Index State Management permissions

See [ISM API]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/).

- cluster:admin/opendistro/ism/managedindex/add
- cluster:admin/opendistro/ism/managedindex/change
- cluster:admin/opendistro/ism/managedindex/remove
- cluster:admin/opendistro/ism/managedindex/explain
- cluster:admin/opendistro/ism/managedindex/retry
- cluster:admin/opendistro/ism/policy/write
- cluster:admin/opendistro/ism/policy/get
- cluster:admin/opendistro/ism/policy/search
- cluster:admin/opendistro/ism/policy/delete

### Index rollups permissions

See [Index rollups API]({{site.url}}{{site.baseurl}}/im-plugin/index-rollups/rollup-api/).

- cluster:admin/opendistro/rollup/index
- cluster:admin/opendistro/rollup/get
- cluster:admin/opendistro/rollup/search
- cluster:admin/opendistro/rollup/delete
- cluster:admin/opendistro/rollup/start
- cluster:admin/opendistro/rollup/stop
- cluster:admin/opendistro/rollup/explain

### Reporting permissions

See [Creating reports with the Dashboards interface]({{site.url}}{{site.baseurl}}/dashboards/reporting/).

- cluster:admin/opendistro/reports/definition/create
- cluster:admin/opendistro/reports/definition/update
- cluster:admin/opendistro/reports/definition/on_demand
- cluster:admin/opendistro/reports/definition/delete
- cluster:admin/opendistro/reports/definition/get
- cluster:admin/opendistro/reports/definition/list
- cluster:admin/opendistro/reports/instance/list
- cluster:admin/opendistro/reports/instance/get
- cluster:admin/opendistro/reports/menu/download

### Transform job permissions

See [Transforms APIs]({{site.url}}{{site.baseurl}}/im-plugin/index-transforms/transforms-apis/)

- cluster:admin/opendistro/transform/index
- cluster:admin/opendistro/transform/get
- cluster:admin/opendistro/transform/preview
- cluster:admin/opendistro/transform/delete
- cluster:admin/opendistro/transform/start
- cluster:admin/opendistro/transform/stop
- cluster:admin/opendistro/transform/explain

### Observability permissions

See [Observability security]({{site.url}}{{site.baseurl}}/observing-your-data/observability-security/).

- cluster:admin/opensearch/observability/create
- cluster:admin/opensearch/observability/update
- cluster:admin/opensearch/observability/delete
- cluster:admin/opensearch/observability/get

### Cross-cluster replication

See [Cross-cluster replication security]({{site.url}}{{site.baseurl}}/tuning-your-cluster/replication-plugin/permissions/).

- cluster:admin/plugins/replication/autofollow/update

### Reindex

See [Reindex document]({{site.url}}{{site.baseurl}}/api-reference/document-apis/reindex/).

- cluster:admin/reindex/rethrottle

### Snapshot repository permissions

See [Snapshot APIs]({{site.url}}{{site.baseurl}}/api-reference/snapshots/index/).

- cluster:admin/repository/delete
- cluster:admin/repository/get
- cluster:admin/repository/put
- cluster:admin/repository/verify

### Reroute

See [Cluster manager task throttling]({{site.url}}{{site.baseurl}}/tuning-your-cluster/cluster-manager-task-throttling/).

- cluster:admin/reroute

### Script permissions

See [Script APIs]({{site.url}}{{site.baseurl}}/api-reference/script-apis/index/).

- cluster:admin/script/delete
- cluster:admin/script/get
- cluster:admin/script/put

### Update settings permission

See [Update settings]({{site.url}}{{site.baseurl}}api-reference/index-apis/update-settings/) on the Index APIs page.

- cluster:admin/settings/update

### Snapshot permissions

See [Snapshot APIs]({{site.url}}{{site.baseurl}}/api-reference/snapshots/index/).

- cluster:admin/snapshot/create
- cluster:admin/snapshot/delete
- cluster:admin/snapshot/get
- cluster:admin/snapshot/restore
- cluster:admin/snapshot/status
- cluster:admin/snapshot/status*

### Task permissions

See [Tasks]({{site.url}}{{site.baseurl}}/api-reference/tasks/) in the API Reference section.

- cluster:admin/tasks/cancel
- cluster:admin/tasks/test
- cluster:admin/tasks/testunblock

### Security Analytics permissions

See [API tools]({{site.url}}{{site.baseurl}}/security-analytics/api-tools/index/).

| **Permission** | **Description** |
| :--- | :--- |
| cluster:admin/opensearch/securityanalytics/alerts/get | Permission to get alerts |
| cluster:admin/opensearch/securityanalytics/alerts/ack | Permission to acknowledge alerts |
| cluster:admin/opensearch/securityanalytics/detector/get | Permission to get detectors |
| cluster:admin/opensearch/securityanalytics/detector/search | Permission to search detectors |
| cluster:admin/opensearch/securityanalytics/detector/write | Permission to create and update detectors |
| cluster:admin/opensearch/securityanalytics/detector/delete | Permission to delete detectors |
| cluster:admin/opensearch/securityanalytics/findings/get | Permission to get findings |
| cluster:admin/opensearch/securityanalytics/mapping/get | Permission to get field mappings by index |
| cluster:admin/opensearch/securityanalytics/mapping/view/get | Permission to get field mappings by index and view mapped and unmapped fields |
| cluster:admin/opensearch/securityanalytics/mapping/create | Permission to create field mappings |
| cluster:admin/opensearch/securityanalytics/mapping/update | Permission to update field mappings |
| cluster:admin/opensearch/securityanalytics/rules/categories | Permission to get all rule categories |
| cluster:admin/opensearch/securityanalytics/rule/write | Permission to create and update rules |
| cluster:admin/opensearch/securityanalytics/rule/search | Permission to search for rules |
| cluster:admin/opensearch/securityanalytics/rules/validate | Permission to validate rules |
| cluster:admin/opensearch/securityanalytics/rule/delete | Permission to delete rules |

### Monitoring permissions

Cluster permissions for monitoring the cluster apply to read-only operations, such as checking cluster health and getting information about usage on nodes or tasks running in the cluster.

See [REST API reference]({{site.url}}{{site.baseurl}}/api-reference/index/).

- cluster:monitor/allocation/explain
- cluster:monitor/health
- cluster:monitor/main
- cluster:monitor/nodes/hot_threads
- cluster:monitor/nodes/info
- cluster:monitor/nodes/liveness
- cluster:monitor/nodes/stats
- cluster:monitor/nodes/usage
- cluster:monitor/remote/info
- cluster:monitor/state
- cluster:monitor/stats
- cluster:monitor/task
- cluster:monitor/task/get
- cluster:monitor/tasks/list

### Index templates

The index template permissions are for indexes but apply globally to the cluster.

See [Index templates]({{site.url}}{{site.baseurl}}/im-plugin/index-templates/).

- indices:admin/index_template/delete
- indices:admin/index_template/get
- indices:admin/index_template/put
- indices:admin/index_template/simulate
- indices:admin/index_template/simulate_index


## Index permissions

These permissions apply to an index or index pattern. You might want a user to have read access to all indices (i.e. `*`), but write access to only a few (e.g. `web-logs` and `product-catalog`).

- indices:admin/aliases
- indices:admin/aliases/exists
- indices:admin/aliases/get
- indices:admin/analyze
- indices:admin/cache/clear
- indices:admin/close
- indices:admin/close*
- indices:admin/create (create indices)
- indices:admin/data_stream/create
- indices:admin/data_stream/delete
- indices:admin/data_stream/get
- indices:admin/delete (delete indices)
- indices:admin/exists
- indices:admin/flush
- indices:admin/flush*
- indices:admin/forcemerge
- indices:admin/get (retrieve index and mapping)
- indices:admin/mapping/put
- indices:admin/mappings/fields/get
- indices:admin/mappings/fields/get*
- indices:admin/mappings/get
- indices:admin/open
- indices:admin/plugins/replication/index/setup/validate
- indices:admin/plugins/replication/index/start
- indices:admin/plugins/replication/index/pause
- indices:admin/plugins/replication/index/resume
- indices:admin/plugins/replication/index/stop
- indices:admin/plugins/replication/index/update
- indices:admin/plugins/replication/index/status_check
- indices:admin/refresh
- indices:admin/refresh*
- indices:admin/resolve/index
- indices:admin/rollover
- indices:admin/seq_no/global_checkpoint_sync
- indices:admin/settings/update
- indices:admin/shards/search_shards
- indices:admin/shrink
- indices:admin/synced_flush
- indices:admin/template/delete
- indices:admin/template/get
- indices:admin/template/put
- indices:admin/types/exists
- indices:admin/upgrade
- indices:admin/validate/query
- indices:data/read/explain
- indices:data/read/field_caps
- indices:data/read/field_caps*
- indices:data/read/get
- indices:data/read/mget
- indices:data/read/mget*
- indices:data/read/msearch
- indices:data/read/msearch/template
- indices:data/read/mtv (multi-term vectors)
- indices:data/read/mtv*
- indices:data/read/plugins/replication/file_chunk
- indices:data/read/plugins/replication/changes
- indices:data/read/scroll
- indices:data/read/scroll/clear
- indices:data/read/search
- indices:data/read/search*
- indices:data/read/search/template
- indices:data/read/tv (term vectors)
- indices:data/write/bulk
- indices:data/write/bulk*
- indices:data/write/delete (delete documents)
- indices:data/write/delete/byquery
- indices:data/write/plugins/replication/changes
- indices:data/write/index (add documents to existing indices)
- indices:data/write/reindex
- indices:data/write/update
- indices:data/write/update/byquery
- indices:monitor/data_stream/stats
- indices:monitor/recovery
- indices:monitor/segments
- indices:monitor/settings/get
- indices:monitor/shard_stores
- indices:monitor/stats
- indices:monitor/upgrade
