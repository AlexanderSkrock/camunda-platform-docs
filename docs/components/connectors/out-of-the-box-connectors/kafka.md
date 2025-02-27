---
id: kafka
title: Kafka Producer Connector
sidebar_label: Kafka Producer Connector
description: The Kafka Producer Connector allows you to connect your BPMN service with Kafka. Learn how to create a Kafka Producer Connector and make it executable.
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

<Tabs groupId="kafka" defaultValue="outbound" queryString values={
[
{label: 'Kafka Producer Connector', value: 'outbound' },
{label: 'Kafka Consumer Connector', value: 'inbound' }
]}>

<TabItem value='outbound'>

The **Kafka Producer Connector** is an outbound Connector that allows you to connect your BPMN service with [Kafka](https://kafka.apache.org/) to produce messages.

## Prerequisites

To use the **Kafka Producer Connector**, you need to have a Kafka instance with configured bootstrap server.
It is highly recommended not to expose your sensitive data as plain text, but rather use Camunda secrets. Follow our documentation on [managing secrets](/components/console/manage-clusters/manage-secrets.md) to learn more.

## Create a Kafka Producer Connector task

To use the **Kafka Producer Connector** in your process, either change the type of existing task by clicking on it and using the wrench-shaped **Change type** context menu icon, or create a new Connector task by using the **Append Connector** context menu. Follow our [guide on using Connectors](/components/connectors/use-connectors/index.md) to learn more.

## Make your Kafka Producer Connector for publishing messages executable

To make your **Kafka Producer Connector** for publishing messages executable, take the following steps:

1. (Optional) Set the relevant credentials in the **Authentication** section. For example, `{{secrets.MY_KAFKA_USERNAME}}`. See the relevant [appendix section](#what-mechanism-is-used-to-authenticate-against-kafka) to find more about Kafka secure authentication.
2. In the **Kafka** section, set the URL of bootstrap server(s); comma-separated if more than one server required.
3. In the **Kafka** section, set the topic name.
4. (Optional) In the **Kafka** section, fill out the field **Additional properties** to set producer configuration values. See the list of supported configurations at the [official Kafka documentation page](https://kafka.apache.org/documentation/#producerconfigs). Also check preconfigured values for the **Kafka Producer Connector** in the relevant [appendix section](#what-are-default-kafka-producer-client-properties).
5. In the **Message** section, set the **Key** and the **Value** that will be sent to Kafka topic.

## Kafka Producer Connector response

The **Kafka Producer Connector** returns metadata for a record that has been acknowledged by the Kafka instance.

The following fields are available in the `response` variable:

- `timestamp`: the timestamp of the message
- `offset`: message offset
- `partition`: message partition
- `topic`: topic name

You can read about these fields at the [official Kafka documentation page](https://kafka.apache.org/documentation/#intro_concepts_and_terms).

You can use an output mapping to map the response:

1. Use **Result Variable** to store the response in a process variable. For example, `myResultVariable`.
2. Use **Result Expression** to map fields from the response into process variables. For example:

```
= {
  "messageAcknowledgedAt": response.timestamp
}
```

## Appendix & FAQ

### What mechanism is used to authenticate against Kafka?

If the fields **Username** and **Password** are not empty, by default the **Kafka Producer Connector** enables the credentials-based SASL SSL authentication and the following properties are set:

```
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule   required username='<Your Username>'   password='<Your Password>';
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
```

If any of the field is not populated, you have to configure your security method in respect to your Kafka configuration. You can do so via the field **Additional properties**.

### What are default Kafka Producer client properties?

- Authentication properties (only if both **Username** and **Password** are not empty):

```
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule   required username='<Your Username>'   password='<Your Password>';
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
```

- Bootstrap server property:

```
bootstrap.servers=<bootstrap server(s) from BPMN>
```

- Message properties:

```
key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.StringSerializer
```

- Miscellaneous properties:

```
session.timeout.ms=45000
client.dns.lookup=use_all_dns_ips
acks=all
delivery.timeout.ms=45000
```

### What is the precedence of client properties loading?

Properties loading consists of three steps:

1. Construct client properties from the BPMN diagram: authentication, bootstrap server, message properties.
2. Load miscellaneous properties.
3. Load and **override** properties from the field **Additional properties**.

### How do I set or override additional client properties?

The following example sets a new client property `client.id` and overrides SASL mechanism to SCRAM SHA-256 instead of plain text:

```
= {
  "client.id":"MyDemoClientId",
  "sasl.mechanism":"SCRAM-SHA-256"
}
```

</TabItem>

<TabItem value='inbound'>

The **Kafka Consumer Connector** allows you to consume messages by subscribing to [Kafka](https://kafka.apache.org/) topics and map them your BPMN processes as start or intermediate events.

## Prerequisites

To use the **Kafka Consumer Connector**, you need to have a Kafka instance with configured bootstrap server.
It is highly recommended not to expose your sensitive data as plain text, but rather use Camunda secrets. Follow our documentation on [managing secrets](/components/console/manage-clusters/manage-secrets.md) to learn more.

## Create a Kafka Consumer Connector task

1. Add a **Start Event** or an **Intermediate Event** to your BPMN diagram to get started.
2. Change its template to a Kafka Consumer.
3. Fill in all required properties.
4. Complete your BPMN diagram.
5. Deploy the diagram to activate the Kafka consumer.

## Make your Kafka Consumer Connector executable

To make your **Kafka Consumer Connector** executable, take the following steps:

1. In the **Authentication** section, select the **Authentication type**.
2. (If you selected _Credentials_ as the **Authentication type**) In the **Authentication** section, set the relevant credentials. For example, `{{secrets.MY_KAFKA_USERNAME}}`. Refer to the relevant [appendix section](#what-mechanism-is-used-to-authenticate-against-kafka) to find more about Kafka secure authentication.
3. In the **Kafka** section, set the URL of bootstrap server(s); comma-separated if more than one server required.
4. In the **Kafka** section, set the topic name.
5. (Optional) In the **Kafka** section, fill out the field **Additional properties** to set consumer configuration values. See the list of supported configurations at the [official Kafka documentation page](https://kafka.apache.org/documentation/#consumerconfigs). Additionally, check preconfigured values for the **Kafka Consumer Connector** in the relevant [appendix section](#what-are-default-kafka-consumer-client-properties).
6. In the **Kafka** section, you can set the **Offsets** for the partition. The number of offsets specified should match the number of partitions on the current topic.
7. In the **Kafka** section, you can set the **Auto offset reset** which tells the Connector what strategy to use when there is no initial offset in Kafka or if the specified offsets do not exist on the server.
8. In the **Activation** section, you can set the **Activation Condition**. Based on this condition, we either start a process instance or do nothing if the condition is not met. For example, `=(value.itemId = "a4f6j2")`. Leave this field empty to trigger your webhook every time.

When using the **Kafka Consumer Connector** with an **Intermediate Catch Event**, fill in the **Correlation key (process)** and **Correlation key (payload)**.

- **Correlation key (process)** is a FEEL expression that defines the correlation key for the subscription. This corresponds to the **Correlation key** property of a regular **Message Intermediate Catch Event**.
- **Correlation key (payload)** is a FEEL expression used to extract the correlation key from the incoming message. This expression is evaluated in the Connector Runtime and the result is used to correlate the message.

For example, given that your correlation key is defined with `myCorrelationKey` process variable, and the value contains `"value":{"correlationKey":"myValue"}`, your correlation key settings will look like this:

- **Correlation key (process)**: `=myCorrelationKey`
- **Correlation key (payload)**: `=value.correlationKey`

Learn more about correlation keys in the [messages guide](../../../concepts/messages).

## Activate the Kafka Consumer Connector by deploying your diagram

Once you click the **Deploy** button, your Kafka Consumer will be activated and starts consuming messages from the specified topic.

## Kafka Consumer Connector response

The **Kafka Consumer Connector** returns the consumed message.

The following fields are available in the `response` variable:

- `key`: The key of the message.
- `value`: The value of the message.
- `rawValue`: The value of the message as a JSON string.

You can use an output mapping to map the response:

1. Use **Result variable** to store the response in a process variable. For example, `myResultVariable`.
2. Use **Result expression** to map fields from the response into process variables. For example:

```
= {
  "itemId": value.itemId
}
```

## Appendix & FAQ

### What mechanism is used to authenticate against Kafka?

If you selected _Credentials_ as **Authentication type** and the fields **Username** and **Password** are not empty, by default the **Kafka Consumer Connector** enables the credentials-based SASL SSL authentication and the following properties are set:

```
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule   required username='<Your Username>'   password='<Your Password>';
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
```

If any of the field is not populated, you must configure your security method in respect to your Kafka configuration. You can do so via the field **Additional properties**.

### What are default Kafka Consumer client properties?

- Authentication properties (only if both **Username** and **Password** are not empty):

```
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule   required username='<Your Username>'   password='<Your Password>';
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
```

- Bootstrap server property:

```
bootstrap.servers=<bootstrap server(s) from BPMN>
```

- Message properties:

```
key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

- Miscellaneous properties:

```
session.timeout.ms=45000
client.dns.lookup=use_all_dns_ips
acks=all
group.id=kafka-inbound-connector-{{bpmnProcessId}}
enable.auto.commit=false
```

### What is the precedence of client properties loading?

Properties loading consists of three steps:

1. Construct client properties from the BPMN diagram: authentication, bootstrap server, message properties.
2. Load miscellaneous properties.
3. Load and **override** properties from the field **Additional properties**.

### How is the message payload deserialized?

Kafka messages usually use JSON format, therefore we first try to deserialize it as a `JsonElement`. If this fails (e.g. because of wrong format) we use the `String` representation of the original raw value. For convenience, we always store the original raw value as `String` in a different attribute.

The deserialized object structure:

```
{
  key: "String"
  rawValue: "String"
  value: {}
}
```

### When is the offset committed? What happens if the Connector execution fails?

The following outcomes are possible:

- If Connector execution is successful and **Activation condition** was met, the offset is committed.
- If **Activation condition** was not met, the offset is also committed to prevent consuming the same message twice.
- If Connector execution fails due to an unexpected error (e.g. Zeebe is unavailable), the offset is not committed.

### What lifecycle does the Kafka Consumer Connector have?

The Kafka Consumer Connector is a long-running Connector that is activated when the process is deployed and deactivated when the process is un-deployed or overwritten by a new version.

</TabItem>

</Tabs>
