# 06 — AWS Lambda: very simple tutorial

AWS Lambda runs your code when something triggers it. AWS manages the servers, so you focus on the function code and configuration.

```text
Trigger -> JSON event -> Lambda function -> result or another AWS service
```

## Main parts

- **Function:** your code and settings.
- **Trigger:** the event source that invokes the function.
- **Event:** input data, commonly represented as JSON.
- **Execution role:** IAM permissions used by the running function.
- **CloudWatch Logs:** a common place for function logs.

## Online-shop example

An order-created event triggers a Lambda function.

1. EventBridge matches an `OrderCreated` event.
2. EventBridge invokes the Lambda function.
3. Lambda starts an execution environment when needed and passes the event to the handler.
4. The code validates the order and writes a summary to an approved destination.
5. The execution role must permit that write.
6. The invocation succeeds or fails; retries and failure handling depend on how it was invoked and configured.

Illustrative Python handler:

```python
def lambda_handler(event, context):
    order_id = event["detail"]["orderId"]
    print(f"Processing order {order_id}")
    return {"processed": order_id}
```

## When to use it

- Short event-driven processing.
- Small APIs and automation.
- Scheduled tasks that do not need a permanently running server.

Consider containers or EC2 when you need a continuously running process, full server control, or a workload that does not fit Lambda's execution model and quotas.

## What to remember

- Lambda is compute, not storage.
- A trigger invokes a function with event data.
- The execution role controls what the function may access.
- Design for retries: repeated delivery must not create harmful duplicate results.
- Monitor errors, duration, throttling, and logs.

## Practice

1. The function must write to DynamoDB. Where is that permission granted?
2. A retry processes the same order twice. What design property is needed?
3. Would Lambda be the natural first choice for a program that must run continuously?

<details>
<summary>Suggested answers</summary>

1. In the function's IAM execution role, scoped to the required actions and table.
2. Idempotency: repeated processing should produce the same safe outcome rather than duplicate effects.
3. Usually no. Evaluate a continuously running compute option such as containers or EC2.

</details>

## Official sources

- [AWS Lambda functions](https://docs.aws.amazon.com/lambda/latest/dg/lambda-functions-chapter.html)
- [Managing permissions in AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-permissions.html)

Checked 13 September 2026. No AWS resources were created.
