# 07 — Amazon EventBridge step-by-step tutorial

## Goal

Create a custom EventBridge event bus and an event-pattern rule that sends only matching order events to a Lambda function. Send one matching event and one non-matching event, verify the routing in CloudWatch Logs, and delete every resource created by the tutorial.

**Difficulty:** Beginner  
**Time:** About 25–30 minutes  
**AWS charges:** Custom events sent to EventBridge can incur event-ingestion charges. The Lambda target can incur request and execution-time charges, and CloudWatch Logs can incur ingestion and storage charges. This lab sends only two small events, but Free Tier coverage is not guaranteed. Complete the cleanup section when finished.

## What you will create

| Item | Tutorial value |
| --- | --- |
| Lambda function | `tutorial-eventbridge-target` |
| Lambda runtime | Latest available Python 3.x runtime |
| Lambda execution role | A new role created automatically by Lambda |
| Lambda log group | `/aws/lambda/tutorial-eventbridge-target` |
| Custom event bus | `tutorial-event-bus` |
| EventBridge rule | `tutorial-ready-order-rule` |
| Matching event order ID | `1001` with status `READY` |
| Non-matching event order ID | `1002` with status `CANCELLED` |

An **event bus** receives events. A **rule** compares each event with an event pattern. A **target** is the resource to which EventBridge sends a matching event. In this tutorial, the target is a Lambda function.

## Part 1 — Create the Lambda target

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the upper-right Region selector, choose your normal learning Region. Use this Region for every resource and CloudShell command in the tutorial.
3. In the top search box, enter **Lambda** and open **Lambda**.
4. In the left navigation pane, choose **Functions**.
5. Choose **Create function**.
6. Select **Author from scratch**.
7. For **Function name**, enter `tutorial-eventbridge-target`.
8. For **Runtime**, choose the latest available **Python 3.x** runtime.
9. For **Architecture**, select **x86_64**.
10. Expand **Change default execution role**.
11. Select **Create a new role with basic Lambda permissions**.
12. Leave the remaining settings at their defaults, then choose **Create function**.

### Record the execution role

1. On the function page, choose **Configuration**.
2. Choose **Permissions**.
3. Under **Execution role**, copy the complete **Role name** into a temporary note.

The role name normally begins with `tutorial-eventbridge-target-role-` and ends with generated characters. You will need the exact name during cleanup.

### Checkpoint

The function state should be **Active**, and its **Permissions** page should show a dedicated execution role.

## Part 2 — Add the Lambda code

1. On the function page, choose the **Code** tab.
2. Under **Code source**, open `lambda_function.py`.
3. Replace all existing code with:

```python
import json


def lambda_handler(event, context):
    print(json.dumps(event))

    return {
        "received": True,
        "orderId": event.get("detail", {}).get("orderId"),
        "status": event.get("detail", {}).get("status")
    }
```

4. Choose **Deploy**.
5. Wait for the message confirming that the function was updated successfully.

The function prints the complete EventBridge event to CloudWatch Logs. It also returns selected values, although EventBridge does not use that return value for this asynchronous invocation.

### Checkpoint

The editor should show the new code, and the deployment message should confirm that the change is live.

## Part 3 — Create a custom event bus

1. In the AWS Console search box, enter **EventBridge** and open **Amazon EventBridge**.
2. In the left navigation pane, choose **Event buses**.
3. Choose **Create event bus**.
4. For **Name**, enter `tutorial-event-bus`.
5. For encryption, keep the default **AWS owned key**.
6. Under **Logs — optional**, remove or turn off every log destination, including **CloudWatch Logs** if it is selected by default.

    This tutorial uses the Lambda function's logs to prove delivery. Event-bus logging would create another log resource that is not needed for this lab.

7. Do not add a dead-letter queue.
8. Leave **Archive** disabled.
9. Leave **Schema discovery** disabled.
10. Do not add a resource-based policy.
11. Choose **Create**.

### Checkpoint

The **Event buses** page should list `tutorial-event-bus`. Open it and confirm that no archive or event-bus log destination was created.

## Part 4 — Create the event-pattern rule

1. In the EventBridge navigation pane, choose **Rules**.
2. Choose **Create rule**.
3. If EventBridge asks you to select a rule builder, choose **Advanced Builder**. This builder lets you enter the event pattern as JSON.
4. For **Name**, enter `tutorial-ready-order-rule`.
5. For **Description**, enter `Routes READY tutorial order events to Lambda.`
6. For **Event bus**, choose `tutorial-event-bus`.
7. For **Rule type**, choose **Rule with an event pattern**.
8. Choose **Next**.
9. For **Event source**, choose **Other** if that choice is shown. If the Console only shows **AWS events or EventBridge partner events**, leave the default selected; the custom JSON pattern in the next step determines what matches.
10. For **Creation method**, choose **Custom pattern (JSON editor)**.
11. Replace the event pattern with:

```json
{
  "source": ["tutorial.orders"],
  "detail-type": ["Order Status Changed"],
  "detail": {
    "status": ["READY"]
  }
}
```

12. Choose **Next**.
13. For **Target type**, choose **AWS service**.
14. For **Select a target**, choose **Lambda function**.
15. For **Function**, choose `tutorial-eventbridge-target`.
16. Keep **Configure target input** set to **Matched events** so that the complete event is sent to Lambda.
17. Do not configure a dead-letter queue.
18. Choose **Next**.
19. On **Configure tags**, choose **Next** without adding a tag.
20. Review the event bus, event pattern, and Lambda target.
21. Choose **Create rule**.

When the rule is created through the Console, EventBridge adds permission to the Lambda function's resource-based policy so that EventBridge can invoke it.

### Checkpoint

Open `tutorial-ready-order-rule` and confirm:

- Rule status is **Enabled**.
- Event bus is `tutorial-event-bus`.
- The pattern requires source `tutorial.orders`, detail type `Order Status Changed`, and status `READY`.
- The target is `tutorial-eventbridge-target`.

Wait about one minute before testing because a new or updated target might not be invoked immediately.

## Part 5 — Send a matching event

1. Choose the **CloudShell** icon in the AWS Console navigation bar.
2. Confirm that CloudShell is using the same Region as `tutorial-event-bus`.
3. Run this command exactly as shown:

```bash
aws events put-events --entries '[
  {
    "Source": "tutorial.orders",
    "DetailType": "Order Status Changed",
    "Detail": "{\"orderId\":\"1001\",\"status\":\"READY\"}",
    "EventBusName": "tutorial-event-bus"
  }
]'
```

**Expected command result:**

```json
{
  "FailedEntryCount": 0,
  "Entries": [
    {
      "EventId": "generated-event-id"
    }
  ]
}
```

Your generated `EventId` will be different. `FailedEntryCount` of `0` proves that EventBridge accepted the event. It does not by itself prove that the rule matched or that Lambda ran.

## Part 6 — Verify the successful route

1. Return to the Lambda Console.
2. Open `tutorial-eventbridge-target`.
3. Choose **Monitor**.
4. Choose **View CloudWatch logs**.
5. Open the newest log stream in `/aws/lambda/tutorial-eventbridge-target`.
6. Find the log entry containing order ID `1001`.

The printed event should contain values similar to:

```json
{
  "source": "tutorial.orders",
  "detail-type": "Order Status Changed",
  "detail": {
    "orderId": "1001",
    "status": "READY"
  }
}
```

EventBridge adds other standard fields, including `version`, `id`, `account`, `time`, `region`, and `resources`.

### Checkpoint

Finding `"orderId": "1001"` in the Lambda log proves this route:

```text
matching event → tutorial-event-bus → tutorial-ready-order-rule → tutorial-eventbridge-target
```

Logs can take several minutes to appear. Refresh the log group if necessary.

## Part 7 — Send a non-matching event

1. Return to CloudShell.
2. Run:

```bash
aws events put-events --entries '[
  {
    "Source": "tutorial.orders",
    "DetailType": "Order Status Changed",
    "Detail": "{\"orderId\":\"1002\",\"status\":\"CANCELLED\"}",
    "EventBusName": "tutorial-event-bus"
  }
]'
```

**Expected command result:** `FailedEntryCount` is `0` and a new `EventId` is returned.

The event bus accepts this event, but the rule should ignore it because the pattern requires `detail.status` to equal `READY`.

### Verify the expected non-match

1. Wait one or two minutes.
2. Return to the CloudWatch log group `/aws/lambda/tutorial-eventbridge-target`.
3. Refresh the log streams and inspect the newest entries.
4. Search for order ID `1002`.

**Expected result:** No Lambda log entry contains `1002`. The event did not match the rule, so EventBridge did not send it to the function.

For additional confirmation, open the EventBridge rule, choose **Monitoring** or **Metrics**, and inspect its invocation metrics after they update. Only the matching event should produce a rule invocation. CloudWatch metrics can take several minutes to update.

## Part 8 — Clean up

Delete the resources in this order so that the rule stops sending events before its bus and target are removed.

### Delete the EventBridge rule

1. Open the EventBridge Console.
2. In the navigation pane, choose **Rules**.
3. For **Event bus**, select `tutorial-event-bus`.
4. Select only `tutorial-ready-order-rule`.
5. Choose **Actions**, then **Delete**.
6. Confirm the deletion.

### Delete the custom event bus

1. In the EventBridge navigation pane, choose **Event buses**.
2. Open `tutorial-event-bus`.
3. Choose **Delete**.
4. Enter any confirmation text requested by the Console, then confirm deletion.

The account's `default` event bus cannot be deleted. Delete only `tutorial-event-bus`.

### Delete the Lambda function

1. Open the Lambda Console and choose **Functions**.
2. Select only `tutorial-eventbridge-target`.
3. Choose **Actions**, then **Delete**.
4. Enter the confirmation text requested by the Console.
5. Choose **Delete**.

### Delete the Lambda log group

Deleting the function does not automatically remove its existing CloudWatch log group.

1. Open the CloudWatch Console.
2. Choose **Logs**, then **Log groups**. In newer layouts, this may appear under **Log Management**.
3. Select only `/aws/lambda/tutorial-eventbridge-target`.
4. Choose **Actions**, then **Delete log group(s)**.
5. Confirm the deletion.

### Delete the dedicated Lambda execution role

IAM roles do not have a separate hourly charge, but this tutorial role is no longer needed.

1. Open the IAM Console.
2. Choose **Roles**.
3. Search for the exact role name recorded in Part 1.
4. Open it and verify that it belongs to `tutorial-eventbridge-target`.
5. Choose **Delete**.
6. Enter the role name if requested, then confirm deletion.

### Final checkpoint

Confirm that all tutorial resources are gone:

- EventBridge no longer lists `tutorial-ready-order-rule` on `tutorial-event-bus`.
- EventBridge no longer lists `tutorial-event-bus`.
- Lambda no longer lists `tutorial-eventbridge-target`.
- CloudWatch Logs no longer lists `/aws/lambda/tutorial-eventbridge-target`.
- IAM no longer lists the exact execution role recorded in Part 1.

## Troubleshooting

### PutEvents returns a failed entry

Confirm that `Source`, `DetailType`, and `Detail` are present, that `Detail` contains a valid JSON object encoded as a string, and that `EventBusName` is exactly `tutorial-event-bus`. Also confirm that CloudShell and the event bus are in the same Region.

### Event 1001 is accepted but Lambda has no log

Wait several minutes and refresh the log group. Then confirm that the rule is enabled, belongs to `tutorial-event-bus`, and targets `tutorial-eventbridge-target`. Check that the event pattern uses the exact case-sensitive values `tutorial.orders`, `Order Status Changed`, and `READY`.

### Event 1002 appears in the Lambda logs

Open the rule and inspect its event pattern. Confirm that the `detail` section restricts `status` to `READY`. If you changed the pattern, save the rule and allow a short time for the update to take effect before sending a new event.

### Access denied appears

Your signed-in identity may not be allowed to create EventBridge resources, call `events:PutEvents`, create or pass the Lambda execution role, add Lambda invocation permission, or view CloudWatch Logs. Use a learning identity with the required permissions, or ask the account administrator to provide them.

### The custom event bus cannot be deleted

Confirm that `tutorial-ready-order-rule` was deleted first. Do not attempt to delete the account's `default` event bus.

## What you learned

- A custom event bus receives application events.
- An event pattern selects events by fields such as `source`, `detail-type`, and values inside `detail`.
- A rule sends only matching events to its configured target.
- EventBridge can invoke a Lambda function asynchronously.
- A successful `PutEvents` response proves event acceptance, while the target's logs prove successful routing and delivery.
- An accepted event can be ignored when it does not match a rule.

## Official AWS documentation

- [Create a custom EventBridge event bus](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-event-bus.html)
- [Create an EventBridge rule using the Advanced Builder](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-wizard.html)
- [Sending events with PutEvents](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-putevents.html)
- [EventBridge event-bus targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html)
- [Delete an EventBridge rule](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-delete-rule.html)
- [Delete a custom event bus](https://docs.aws.amazon.com/eventbridge/latest/userguide/event-bus-delete.html)
- [Amazon EventBridge pricing](https://aws.amazon.com/eventbridge/pricing/)

Instructions checked on 13 September 2026. AWS Console labels can change slightly over time.
