---
id: elasticsearch-exporter
title: "Elasticsearch exporter"
sidebar_label: "Elasticsearch"
description: "The Zeebe Elasticsearch exporter acts as a bridge between Zeebe and Elasticsearch."
---

:::note
Please refer to [Supported Environments](/reference/supported-environments.md#camunda-platform-8-self-managed) to find out which versions of Elasticsearch are supported in a Camunda 8 Self-Managed setup.
:::

The Zeebe Elasticsearch exporter acts as a bridge between
[Zeebe](https://zeebe.io/) and [Elasticsearch](https://www.elastic.co/products/elasticsearch) by
exporting records written to Zeebe streams as documents into several indices.

## Concept

The exporter operates on the idea that it should perform as little as possible on the Zeebe side of
things. In other words, you can think of the indexes into which the records are exported as a
staging data warehouse. Any enrichment or transformation on the exported data should be performed by
your own ETL jobs.

When configured to do so, the exporter will automatically create an index per record value type (see the value type in the Zeebe protocol). Each of these indexes has a
corresponding pre-defined mapping to facilitate data ingestion for your own ETL jobs. You can find
those as templates in this module's resources folder.

:::note
The indexes are created as required, and will not be created twice if they already exist. However, once disabled, they will not be deleted (that is up to the administrator.) Similarly, data is never deleted by the exporter, and must be deleted by the administrator when it is safe to do so.
A [retention](#retention) policy can be configured to automatically delete data after a certain number of days.
:::

## Configuration

:::note
As the exporter is packaged with Zeebe, it is not necessary to specify a `jarPath`.
:::

The exporter can be enabled by configuring it with the `classpath` in the broker settings.

For example:

```yaml
exporters:
  elasticsearch:
    className: io.camunda.zeebe.exporter.ElasticsearchExporter
    args:
    # Refer to the table below for the available args options
```

The exporter can be configured by providing `args`. The table below explains all the different
options, and the default values for these options:

| Option           | Description                                                                              | Default                 |
| ---------------- | ---------------------------------------------------------------------------------------- | ----------------------- |
| url              | Valid URLs as comma-separated string                                                     | `http://localhost:9200` |
| requestTimeoutMs | Request timeout (in ms) for the Elasticsearch. client                                    | `30000`                 |
| index            | Refer to [Index](#index) for the index configuration options.                            |                         |
| bulk             | Refer to [Bulk](#bulk) for the bulk configuration options.                               |                         |
| retention        | Refer to [Retention](#retention) for the retention configuration options                 |                         |
| authentication   | Refer to [Authentication](#authentication) for the authentication configuration options. |                         |

### Index

In most cases, you will not be interested in exporting every single record produced by a Zeebe
cluster, but rather only a subset of them. This can also be configured to limit the kinds of records
being exported (e.g. only events, no commands), and the value type of these records (e.g. only job
and process values).

| Option                        | Description                                                                                             | Default      |
| ----------------------------- | ------------------------------------------------------------------------------------------------------- | ------------ |
| prefix                        | This prefix will be appended to every index created by the exporter; must not contain `_` (underscore). | zeebe-record |
| createTemplate                | If `true` missing indexes will be created automatically.                                                | `true`       |
| command                       | If `true` command records will be exported                                                              | `false`      |
| event                         | If `true` event records will be exported                                                                | `true`       |
| rejection                     | If `true` rejection records will be exported                                                            | `false`      |
| checkpoint                    | If `true` records related to checkpoints will be exported                                               | `false`      |
| commandDistribution           | If `true` records related to command distributions will be exported                                     | `true`       |
| decision                      | If `true` records related to decisions will be exported                                                 | `true`       |
| decisionEvaluation            | If `true` records related to decision evaluations will be exported                                      | `true`       |
| decisionRequirements          | If `true` records related to decisionRequirements will be exported                                      | `true`       |
| deployment                    | If `true` records related to deployments will be exported                                               | `true`       |
| deploymentDistribution        | If `true` records related to deployment distributions will be exported                                  | `true`       |
| error                         | If `true` records related to errors will be exported                                                    | `true`       |
| escalation                    | If `true` records related to escalations will be exported                                               | `true`       |
| incident                      | If `true` records related to incidents will be exported                                                 | `true`       |
| job                           | If `true` records related to jobs will be exported                                                      | `true`       |
| jobBatch                      | If `true` records related to job batches will be exported                                               | `false`      |
| message                       | If `true` records related to messages will be exported                                                  | `true`       |
| messageSubscription           | If `true` records related to message subscriptions will be exported                                     | `true`       |
| messageStartEventSubscription | If `true` records related to message start event subscriptions will be exported                         | `true`       |
| process                       | If `true` records related to processes will be exported                                                 | `true`       |
| processEvent                  | If `true` records related to process events will be exported                                            | `false`      |
| processInstance               | If `true` records related to process instances will be exported                                         | `true`       |
| processInstanceBatch          | If `true` records related to process instances batches will be exported                                 | `false`      |
| processInstanceCreation       | If `true` records related to process instance creations will be exported                                | `true`       |
| processInstanceModification   | If `true` records related to process instance modifications will be exported                            | `true`       |
| processMessageSubscription    | If `true` records related to process message subscriptions will be exported                             | `true`       |
| resourceDeletion              | If `true` records related to resource deletions will be exported                                        | `true`       |
| signal                        | If `true` records related to signals will be exported                                                   | `true`       |
| signalSubscription            | If `true` records related to signal subscriptions will be exported                                      | `true`       |
| timer                         | If `true` records related to timers will be exported                                                    | `true`       |
| variable                      | If `true` records related to variables will be exported                                                 | `true`       |
| variableDocument              | If `true` records related to variable documents will be exported                                        | `true`       |

### Bulk

To avoid too many expensive requests to the Elasticsearch cluster, the exporter performs batch
updates by default. The size of the batch, along with how often it should be flushed (regardless of
size) can be controlled by configuration.

| Option      | Description                                                                                                                                                    | Default            |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ |
| delay       | Delay, in seconds, before force flush of the current batch. This ensures that even when we have low traffic of records, we still export every once in a while. | `5`                |
| size        | The amount of records a batch should have before we flush the batch                                                                                            | `1000`             |
| memoryLimit | The size of the batch, in bytes, before we flush the batch                                                                                                     | `10485760` (10 MB) |

With the default configuration, the exporter will aggregate records and flush them to Elasticsearch:

1. When it has aggregated 1000 records.
2. When the batch memory size exceeds 10 MB.
3. Five seconds have elapsed since the last flush (regardless of how many records were aggregated).

### Retention

A retention policy can be set up to delete old data.
When enabled, this creates an Index Lifecycle Management (ILM) Policy that deletes the data after the specified `minimumAge`.
All index templates created by this exporter apply the created ILM Policy.

| Option     | Description                                                                  | Default                         |
| ---------- | ---------------------------------------------------------------------------- | ------------------------------- |
| enabled    | If `true` the ILM Policy is created and applied to the index templates       | `false`                         |
| minimumAge | Specifies how old the data must be, before the data is deleted as a duration | `30d`                           |
| policyName | The name of the created and applied ILM policy                               | `zeebe-record-retention-policy` |

:::note
The duration can be specified in days `d`, hours `h`, minutes `m`, seconds `s`, milliseconds `ms`, and/or nanoseconds `nanos`.
:::

### Authentication

Providing these authentication options will enable Basic Authentication on the exporter.

| Option   | Description                   | Default |
| -------- | ----------------------------- | ------- |
| username | Username used to authenticate | N/A     |
| password | Password used to authenticate | N/A     |

## Example

Here is an example configuration of the exporter:

```yaml
---
exporters:
  elasticsearch:
    # Elasticsearch Exporter ----------
    # An example configuration for the elasticsearch exporter:
    #
    # These setting can also be overridden using the environment variables "ZEEBE_BROKER_EXPORTERS_ELASTICSEARCH_..."
    #

    className: io.camunda.zeebe.exporter.ElasticsearchExporter

    args:
      # A comma separated list of URLs pointing to the Elasticsearch instances you wish to export to.
      # For example, if you want to connect to multiple nodes for redundancy:
      # url: http://localhost:9200,http://localhost:9201
      url: http://localhost:9200

      bulk:
        delay: 5
        size: 1000
        memoryLimit: 10485760

      retention:
        enabled: true
        minimumAge: 30d
        policyName: zeebe-records-retention-policy

      authentication:
        username: elastic
        password: changeme

      index:
        prefix: zeebe-record
        createTemplate: true

        command: false
        event: true
        rejection: false

        commandDistribution: true
        decisionRequirements: true
        decision: true
        decisionEvaluation: true
        deployment: true
        deploymentDistribution: true
        error: true
        escalation: true
        incident: true
        job: true
        jobBatch: false
        message: true
        messageStartSubscription: true
        messageSubscription: true
        process: true
        processEvent: false
        processInstance: true
        processInstanceCreation: true
        processInstanceModification: true
        processMessageSubscription: true
        resourceDeletion: true
        signal: true
        signalSubscription: true
        timer: true
        variable: true
        variableDocument: true
```
