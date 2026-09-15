# 06 — AWS Lambda step-by-step tutorial

## Goal

Create a small Python Lambda function, invoke it with a successful test event, deliberately invoke it with invalid input, inspect both results in CloudWatch Logs, and delete every resource created by the tutorial.

**Difficulty:** Beginner  
**Time:** About 15–20 minutes  
**AWS charges:** Lambda charges for requests and execution time, and CloudWatch Logs can charge for log ingestion and storage. These few short tests may fit within your account's available Free Tier allowance, but the Free Tier is not guaranteed. Complete the cleanup section when finished.

## What you will create

| Item | Tutorial value |
| --- | --- |
| Lambda function | `tutorial-hello-lambda` |
| Runtime | Latest available Python 3.x runtime |
| Architecture | `x86_64` |
| Execution role | A new role created automatically by Lambda |
| CloudWatch log group | `/aws/lambda/tutorial-hello-lambda` |
| Successful test event | `tutorial-success` |
| Failed test event | `tutorial-failure` |

A Lambda function is code that AWS runs when it receives an event. The execution role is the IAM role that the function assumes while it runs. In this tutorial, the role allows the function to write logs to CloudWatch Logs.

## Part 1 — Create the Lambda function

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the upper-right Region selector, choose your normal learning Region.
3. In the top search box, enter **Lambda** and open **Lambda**.
4. In the left navigation pane, choose **Functions**.
5. Choose **Create function**.
6. Select **Author from scratch**.
7. For **Function name**, enter `tutorial-hello-lambda`.
8. For **Runtime**, choose the latest available **Python 3.x** runtime.
9. For **Architecture**, select **x86_64**.
10. Expand **Change default execution role**.
11. Select **Create a new role with basic Lambda permissions**.
12. Leave the remaining settings at their defaults. Do not create a function URL.
13. Choose **Create function**.

Lambda creates the function and a dedicated execution role. The role name usually begins with `tutorial-hello-lambda-role-` and ends with generated characters.

### Checkpoint

The function page should display a green success message, and the function state should be **Active**.

## Part 2 — Record the execution role name

Record the exact role name now so that you can remove it during cleanup.

1. On the function page, choose the **Configuration** tab.
2. In the left menu, choose **Permissions**.
3. Under **Execution role**, find **Role name**.
4. Copy the complete role name into a temporary note.

tutorial-hello-lambda-role-xdz7gu2f


Only delete this exact role during cleanup. Do not delete another role with a similar name.

## Part 3 — Add and deploy the Python code

1. Choose the **Code** tab.
2. In the **Code source** editor, open `lambda_function.py`.
3. Replace all existing code with:

```python
import json


def lambda_handler(event, context):
    name = event.get("name")

    if not name:
        raise ValueError("The event must contain a non-empty 'name' value.")

    message = f"Hello, {name}!"
    print(message)

    return {
        "statusCode": 200,
        "body": json.dumps({"message": message})
    }
```

4. Choose **Deploy**.
5. Wait for the message confirming that the function was updated successfully.

The handler `lambda_handler` receives the test event as a Python dictionary. The code reads its `name` value, prints a message to the logs, and returns a response. It raises an error when `name` is missing or empty.

### Checkpoint

The editor should show the new code, and the deployment message should confirm that the changes are live.

## Part 4 — Run a successful test

1. Choose the **Test** tab.
2. Under **Test event action**, choose **Create new event** if it is not already selected.
3. For **Event name**, enter `tutorial-success`.
4. Keep **Event sharing settings** set to **Private**.
5. Replace the example event JSON with:

```json
{
  "name": "Tom"
}
```

6. Choose **Save**.
7. Choose **Test**.
8. Expand **Execution result** and open **Details** if necessary.

**Expected result:** The invocation succeeds and the response contains:

```json
{
  "statusCode": 200,
  "body": "{\"message\": \"Hello, Tom!\"}"
}
```

The `body` is shown as an escaped JSON string because the function uses `json.dumps` to turn the inner object into text.

## Part 5 — Run an intentional failure test

This test proves how Lambda reports an unhandled code error.

1. Stay on the **Test** tab.
2. Choose **Create new event**.
3. For **Event name**, enter `tutorial-failure`.
4. Keep the event **Private**.
5. Enter an empty JSON object:

```json
{}
```

6. Choose **Save**.
7. Choose **Test**.
8. Expand **Execution result** and inspect **Details**.

**Expected result:** The invocation fails with a `ValueError` and a message similar to:

```text
The event must contain a non-empty 'name' value.
```

This failure is intentional. The function is validating its input correctly.

## Part 6 — Inspect the CloudWatch logs

1. On the function page, choose the **Monitor** tab.
2. Choose **View CloudWatch logs**. A new CloudWatch page opens.
3. Confirm that the log group name is `/aws/lambda/tutorial-hello-lambda`.
4. Under **Log streams**, open the most recent stream.
5. Find the log entry containing `Hello, Tom!` from the successful test.
6. Find the `ValueError` and stack trace from the failed test.
7. Notice the Lambda-generated `START`, `END`, and `REPORT` entries around each invocation.

### Checkpoint

The log stream should contain evidence of both tests:

- `Hello, Tom!` for the successful invocation.
- `ValueError` for the intentional failure.

New logs can take several minutes to appear. Refresh the page if the newest invocation is not visible yet.

## Part 7 — Clean up

Delete the resources in this order. The execution role name copied in Part 2 is required for the final step.

### Delete the Lambda function

1. Return to the Lambda Console and choose **Functions**.
2. Select only `tutorial-hello-lambda`.
3. Choose **Actions**, then **Delete**.
4. Enter the confirmation text requested by the Console.
5. Choose **Delete**.

### Delete the CloudWatch log group

Deleting a Lambda function does not automatically remove its existing CloudWatch log group.

1. In the AWS Console search box, enter **CloudWatch** and open **CloudWatch**.
2. In the left navigation pane, choose **Logs**, then **Log groups**. In newer Console layouts, this may appear under **Log Management**.
3. Find and select only `/aws/lambda/tutorial-hello-lambda`.
4. Choose **Actions**, then **Delete log group(s)**.
5. Confirm the deletion.

Deleting a log group permanently deletes the log events stored inside it.

### Delete the dedicated execution role

IAM roles do not have a separate hourly charge, but remove this tutorial role because it is no longer needed.

1. In the AWS Console search box, enter **IAM** and open **IAM**.
2. In the left navigation pane, choose **Roles**.
3. Search for the exact role name that you copied in Part 2.
4. Open the role and verify that it belongs to `tutorial-hello-lambda`.
5. Choose **Delete**.
6. Enter the role name if the Console requests it, then confirm deletion.

### Final checkpoint

Confirm that all three resources are gone:

- Lambda no longer lists `tutorial-hello-lambda`.
- CloudWatch Logs no longer lists `/aws/lambda/tutorial-hello-lambda`.
- IAM no longer lists the exact execution role recorded in Part 2.

## Troubleshooting

### The test still returns the original Hello from Lambda response

The new code was saved in the editor but not deployed. Return to the **Code** tab, confirm the code is present, and choose **Deploy** before testing again.

### Runtime.HandlerNotFound appears

Confirm that the file is named `lambda_function.py` and that the function inside it is named `lambda_handler`. Under **Runtime settings**, the handler should be `lambda_function.lambda_handler`.

### The test event cannot be saved

Event names cannot contain spaces. Use `tutorial-success` or `tutorial-failure`, and confirm that the event text is valid JSON.

### No CloudWatch logs appear

Wait several minutes and refresh the log group. If it remains empty, open the function's **Configuration** > **Permissions** page and confirm that the execution role has basic Lambda logging permissions.

### Access denied appears while creating the function

Your signed-in identity may not be allowed to create Lambda functions, create or pass IAM roles, or write logs. Use a learning identity that has the required permissions, or ask the administrator of the account to provide them.

### The IAM role cannot be deleted

Confirm that the Lambda function was deleted first and that you selected the exact dedicated role recorded in Part 2. Do not remove policies from or delete a shared role.

## What you learned

- Lambda runs code in response to an event without requiring you to manage a server.
- A test event becomes the function's `event` input.
- Code changes must be deployed before Lambda runs them.
- The execution role gives the function permission to call other AWS services.
- Successful output and unhandled errors are visible in the Lambda test result and CloudWatch Logs.
- Lambda automatically creates log streams, but deleting the function does not automatically delete its existing log group or execution role.

## Official AWS documentation

- [Create a Lambda function with Python](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html)
- [Testing Lambda functions in the Console](https://docs.aws.amazon.com/lambda/latest/dg/testing-functions.html)
- [View Lambda logs in CloudWatch Logs](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs-view.html)
- [Lambda execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
- [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/)

Instructions checked on 13 September 2026. AWS Console labels can change slightly over time.
