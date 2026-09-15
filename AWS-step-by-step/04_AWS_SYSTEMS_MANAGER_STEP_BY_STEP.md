# 04 — AWS Systems Manager step-by-step tutorial

## Goal

Use AWS Systems Manager Parameter Store to create a configuration value, retrieve it, update it, view its version history, and delete it.

**Difficulty:** Beginner  
**Time:** About 10–15 minutes  
**AWS charges:** Standard Parameter Store parameters using standard throughput have no additional charge. Do not select the Advanced tier or higher throughput for this tutorial.

## What you will create

| Resource | Tutorial value |
| --- | --- |
| Parameter name | `/tutorial/myapp/environment` |
| Initial value | `development` |
| Updated value | `testing` |
| Tier | Standard |
| Type | String |

A Parameter Store parameter is a named key-value pair. Applications and automation can retrieve the value by using its name instead of storing the value directly in code.

## Part 1 — Create the parameter

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the upper-right Region selector, choose your normal learning Region.
3. In the top search box, enter **Systems Manager** and open **AWS Systems Manager**.
4. In the left navigation pane, choose **Parameter Store**. If it is not visible, enter `Parameter Store` in the Systems Manager navigation search box.
5. Choose **Create parameter**.
6. For **Name**, enter:

    `/tutorial/myapp/environment`

7. For **Description**, enter:

    `Application environment used by the Systems Manager tutorial.`

8. For **Tier**, select **Standard**.
9. For **Type**, select **String**.
10. For **Data type**, keep **text**.
11. For **Value**, enter:

    `development`

12. Do not enter a password, token, or other secret. A `String` value is not encrypted as a secret.
13. Under **Tags**, add:

    - Key: `Purpose`
    - Value: `Tutorial`

14. Choose **Create parameter**.

### Checkpoint

On the **My parameters** tab, find `/tutorial/myapp/environment`. Open it and confirm:

- Tier is `Standard`.
- Type is `String`.
- Value is `development`.
- Version is `1`.

The parameter exists only in the AWS Region in which you created it. Its leading `/` characters organize it into the hierarchy `tutorial/myapp`.

## Part 2 — Retrieve the parameter

### Retrieve it in the Console

1. Return to **Parameter Store**.
2. On the **My parameters** tab, choose `/tutorial/myapp/environment`.
3. On the **Overview** tab, locate **Value**.

**Expected result:** The value is `development`.

### Retrieve it with CloudShell

1. Choose the **CloudShell** icon in the AWS Console navigation bar.
2. Confirm CloudShell is using the same Region as the parameter.
3. Run:

```bash
aws ssm get-parameter --name "/tutorial/myapp/environment" --query "Parameter.Value" --output text
```

**Expected result:**

```text
development
```

Now test a name that does not exist:

```bash
aws ssm get-parameter --name "/tutorial/myapp/does-not-exist"
```

**Expected result:** AWS returns a `ParameterNotFound` error. Parameter names must match exactly and are case-sensitive.

## Part 3 — Update the parameter

1. Return to **Systems Manager**, then choose **Parameter Store**.
2. Choose `/tutorial/myapp/environment`.
3. Choose **Edit**.
4. Change **Value** from `development` to:

    `testing`

5. Keep the parameter on the **Standard** tier and keep its type as **String**.
6. Choose **Save changes**.
7. Open the parameter again if the Console returns you to the list.

### Checkpoint

On the **Overview** tab, confirm:

- Value is now `testing`.
- Version is now `2`.

Choose the **History** tab. You should see:

| Version | Value |
| --- | --- |
| 1 | `development` |
| 2 | `testing` |

Each saved change creates a new version. Retrieving the parameter without specifying a version returns the latest version.

## Part 4 — Verify the latest value

1. Open CloudShell again.
2. Run the same retrieval command:

```bash
aws ssm get-parameter --name "/tutorial/myapp/environment" --query "Parameter.Value" --output text
```

**Expected result:**

```text
testing
```

This proves that an application retrieving the parameter by name can receive the new value without changing the parameter name.

## Part 5 — Clean up

1. In the Systems Manager navigation pane, choose **Parameter Store**.
2. On the **My parameters** tab, select the checkbox beside `/tutorial/myapp/environment`.
3. Choose **Delete**.
4. In the confirmation dialog, choose **Delete parameters**.

Deleting a parameter removes all of its versions. They cannot be restored. AWS requires a short wait before a parameter with the same name can be created again.

### Final checkpoint

Search for `/tutorial/myapp/environment` on the **My parameters** tab.

**Expected result:** The parameter is no longer listed.

You can also run:

```bash
aws ssm get-parameter --name "/tutorial/myapp/environment"
```

**Expected result:** `ParameterNotFound`.

## Troubleshooting

### ParameterNotFound appears during the first retrieval

Confirm the name includes the leading `/`, uses the same capitalization, and that CloudShell and Parameter Store are using the same Region.

### AccessDeniedException appears

The signed-in identity does not have the required Parameter Store permission. Creating and editing use `ssm:PutParameter`; retrieving uses `ssm:GetParameter`; history uses `ssm:GetParameterHistory`; and deletion uses `ssm:DeleteParameter`.

### The parameter became Advanced

Do not continue creating advanced versions. An advanced parameter cannot be changed back to Standard. Delete it and recreate it as a Standard parameter if you do not need advanced features.

## What you learned

- Parameter Store keeps configuration as named key-value pairs.
- Parameter hierarchy paths help organize related values.
- Standard `String` parameters are suitable for non-sensitive configuration.
- Updating a parameter creates a new numbered version.
- Retrieval without a version returns the latest value.
- Parameters are Regional and IAM controls who can read or change them.

## Official AWS documentation

- [Create a Parameter Store parameter in the Console](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-create-console.html)
- [Working with parameter versions](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-versions.html)
- [Delete parameters from Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/deleting-parameters.html)
- [AWS Systems Manager pricing](https://aws.amazon.com/systems-manager/pricing/)

Instructions checked on 13 September 2026. AWS Console labels can change slightly over time.
