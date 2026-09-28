---
layout: post
title: "Spring Boot and ActiveMQ Classic: choose queue names that stay whole"
date: 2026-09-28 22:38:00 +0200
categories: spring-boot activemq messaging
---

A queue name should be an identifier that every part of a messaging application interprets the same way. A name such as `billing.invoice.created` looks readable, but dots can become ambiguous when a destination is copied into configuration keys, parsed by application code, or passed through tools that treat dots as path separators.

In a Spring Boot application using ActiveMQ Classic, that ambiguity can show up as a consumer configured with only part of the intended name. The consumer then listens to the wrong destination and receives no messages from the queue the producer uses. A later request may appear to work if it supplies or resolves the complete name, making the issue look intermittent.

## A dot is not inherently invalid

ActiveMQ Classic supports dots in destination names. A queue named `billing.invoice.created` is not automatically split into `billing` and `invoice.created` by the broker. If a consumer appears to use only part of a dotted name, look for the place where the destination is configured or transformed: property binding, string splitting, custom destination-building code, or an external tool.

For example, dots are commonly used to separate nested keys in Spring configuration. That is useful for a property path such as `app.messaging.queue`, but it can be confusing if destination names themselves are also treated as paths. The queue value and the configuration key are different things; keep them separate and pass the complete configured value to both producers and consumers.

## Prefer a simple, consistent convention

For queue names that need to work across application code, configuration, deployment tools, and monitoring, use lowercase ASCII letters and digits with hyphens between words. Underscores are also a reasonable separator if they are already standard in your system. Avoid whitespace and punctuation that another layer might interpret specially, including dots, colons, slashes, and wildcard characters.

A useful pattern is:

`<application>-<domain>-<event-or-purpose>-v<version>`

For example:

`orders-order-created-v1`

The version is optional, but can make intentional contract changes easier to roll out. Include an environment prefix only if environments share a broker; separate brokers generally do not need queue names like `prod-orders-order-created-v1`.

## Keep configuration and consumers aligned

Store the destination as a configuration value, rather than embedding or reconstructing it in several places:

```yaml
app:
  messaging:
    queues:
      order-created: orders-order-created-v1
```

Use that same value for the listener and the sender:

```java
@JmsListener(destination = "${app.messaging.queues.order-created}")
public void receive(OrderCreated event) {
    // Handle the event.
}
```

```java
@Value("${app.messaging.queues.order-created}")
private String orderCreatedQueue;

public void publish(OrderCreated event) {
    jmsTemplate.convertAndSend(orderCreatedQueue, event);
}
```

In a larger application, bind the messaging settings to a typed configuration class rather than repeating property expressions. That gives the destination one source of truth and makes configuration validation easier.

## Diagnose a partial destination

If a consumer listens to only part of the expected name, log the resolved destination at startup and log the destination used when sending. Compare the exact strings, including case and whitespace. Then check that no code calls `split("\\.")`, truncates the value, or builds a destination from a property path. Confirm the resolved Spring configuration for the active profile and inspect the actual destinations in the broker console or JMX.

Do not rely on a second receive attempt as a fix. A message sent to one queue is not delivered to a differently named queue, and retrying with a partial name can hide the original mismatch rather than resolve it.

Dots can be perfectly valid in an ActiveMQ Classic queue name. Still, a straightforward convention such as lowercase kebab-case reduces ambiguity at integration boundaries and makes it easier to ensure that every producer and consumer uses the exact same destination.
