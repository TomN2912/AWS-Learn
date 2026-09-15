# 07 — Amazon EventBridge: very simple tutorial

Amazon EventBridge routes events from producers to consumers. It helps systems react to changes without the producer calling every consumer directly.

## Main flow

```text
Event producer -> event bus -> rule matches event -> target receives event
```

- **Event:** a JSON message describing something that happened.
- **Event bus:** receives events.
- **Rule:** matches selected events using an event pattern.
- **Target:** the destination, such as Lambda, SQS, SNS, or Step Functions.

EventBridge also includes **Scheduler** for invoking targets at specified times and **Pipes** for point-to-point integrations with optional filtering and transformation. This lesson focuses on event buses and rules.

## Online-shop example

The order service publishes this illustrative event:

```json
{
  "source": "example.shop.orders",
  "detail-type": "OrderCreated",
  "detail": { "orderId": "A123", "total": 65.00 }
}
```

A rule matches:

```json
{
  "source": ["example.shop.orders"],
  "detail-type": ["OrderCreated"]
}
```

The rule sends matching events to an order-processing Lambda function. Events that do not match are not sent by this rule.

Configure target permissions, retries, and a dead-letter queue where appropriate. Consumers should handle duplicate deliveries safely. EventBridge routing does not by itself create a complete business transaction across every target.

## When to use it

- React to AWS service or application events.
- Connect loosely coupled services.
- Route different event types to different consumers.

Use Amazon SQS when the central requirement is a durable work queue that consumers poll. Use Amazon SNS for straightforward push fan-out. The services can also be combined.

## What to remember

- Producer creates an event.
- Event bus receives it.
- Rule filters it.
- Target processes it.
- Design targets for failure, retry, and possible duplicates.

## Practice

1. Which part selects only `OrderCreated` events?
2. What happens to an event that does not match this rule?
3. Why should the Lambda target be safe if it receives the same event again?

<details>
<summary>Suggested answers</summary>

1. The rule's event pattern.
2. This rule does not send it to its target; another rule could still match it.
3. Delivery and retries can produce repeated processing, so the consumer should avoid duplicate business effects.

</details>

## Official source

- [Event bus targets in Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html)

Checked 13 September 2026. No AWS resources were created.
