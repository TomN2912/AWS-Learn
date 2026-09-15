# 03 — AWS Config step-by-step tutorial

## Goal

Enable AWS Config with a narrow recording scope, add an AWS managed rule that checks whether EBS encryption by default is enabled, view the compliance result, and clean up the lab.

**Difficulty:** Beginner  
**Time:** About 20–30 minutes  
**AWS charges:** Yes. AWS Config charges for recorded configuration items and rule evaluations. The delivery S3 bucket can also incur small storage charges. Cleanup is included.

## What you will create

| Resource | Tutorial value |
| --- | --- |
| AWS Config customer managed recorder | Console default name, normally `default` |
| Delivery channel | Console default name, normally `default` |
| S3 delivery bucket | `tutorial-aws-config-<unique-suffix>` |
| AWS Config managed rule | `tutorial-ebs-encryption-default` |

The rule checks a Regional account setting. You do not need to create an EBS volume. A `COMPLIANT` or `NON_COMPLIANT` result is acceptable for this tutorial.

## Part 1 — Set up AWS Config

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the upper-right Region selector, choose the Region in which you completed your EBS tutorial, or choose your normal learning Region.
3. In the top search box, enter **AWS Config** and open **AWS Config**.
4. Look at the page that appears:

    - If you see **Get started**, continue with step 5.
    - If AWS Config is already recording resources, do not replace its existing recorder or delivery settings. Skip to **Part 2** and remember that cleanup must remove only the tutorial rule.

5. Choose **Get started**.
6. Under **Recording method**, select **Specific resource types**.
7. Add the resource type **AWS EC2 Volume** (`AWS::EC2::Volume`) and choose **Continuous recording**.
8. Add **AWS Config Resource Compliance** (`AWS::Config::ResourceCompliance`) and choose **Continuous recording**. This lets Config record rule-compliance results accurately.
9. Under **Data governance**, set the retention period to `30 days`.
10. For **IAM role for AWS Config**, choose the AWS Config **service-linked role** option.
11. Under **Delivery method**, choose **Create a bucket**.
12. Enter a globally unique bucket name using this pattern:

    `tutorial-aws-config-<unique-suffix>`

    Replace `<unique-suffix>` with lowercase letters and numbers, for example `tutorial-aws-config-tn-20260913-a`. Record your actual name here:

    `tutorial-aws-config-124____________________________________________`

13. Leave the optional Amazon SNS notification setting turned off.
14. Choose **Next**.
15. On the initial **Rules** page, do not select a rule yet. Choose **Next**.
16. Review the recording scope, role, and S3 bucket carefully.
17. Choose **Confirm** to finish setup.

### Checkpoint

Open **Settings** in the AWS Config navigation pane. On the **Customer managed recorder** tab, confirm that:

- Recording is on.
- The recording strategy is specific resource types.
- `AWS::EC2::Volume` is included.
- `AWS::Config::ResourceCompliance` is included.

The configuration recorder now captures supported changes in its configured scope. The delivery channel sends configuration history and snapshots to the S3 bucket.

## Part 2 — Add the managed rule

1. In the AWS Config navigation pane, choose **Rules**.
2. Choose **Add rule**.
3. In the managed-rule search box, enter:

    `ec2-ebs-encryption-by-default`

4. Select the AWS managed rule named **ec2-ebs-encryption-by-default**.
5. Choose **Next**.
6. For **Name**, enter `tutorial-ebs-encryption-default`.
7. Keep **Detective evaluation** enabled. This managed rule uses a periodic trigger.
8. The rule has no parameters, so leave the parameters area unchanged.
9. Choose **Save** or **Add rule**, depending on the label displayed by the Console.

### Checkpoint

The **Rules** page should show `tutorial-ebs-encryption-default`. Its status may initially display `Evaluating` or `Insufficient data` while AWS Config performs the first evaluation.

## Part 3 — View the compliance result

1. Open the rule `tutorial-ebs-encryption-default`.
2. Wait for the initial evaluation to finish, then refresh the page.
3. If a result does not appear, choose **Actions**, then **Re-evaluate**. Wait briefly and refresh again.
4. Read the result:

    - `COMPLIANT` means EBS encryption by default is enabled in this Region.
    - `NON_COMPLIANT` means EBS encryption by default is disabled in this Region.

Do not change the EBS encryption setting for this lab. The purpose is to observe how AWS Config reports the current setting.

### Checkpoint

You have now used the complete AWS Config flow:

```text
Configuration recorder -> AWS Config rule -> compliance evaluation
```

The managed rule detects the setting; it does not automatically change the setting or prevent resource creation.

## Part 4 — Clean up

### Delete the tutorial rule

1. In AWS Config, choose **Rules**.
2. Open `tutorial-ebs-encryption-default`.
3. Choose **Actions**, then **Delete rule**.
4. Enter `Delete` exactly as requested and confirm.

If AWS Config was already configured before this tutorial, stop here. Do not stop or delete a recorder, delivery channel, or S3 bucket that belongs to an existing environment.

Continue below only if you created the AWS Config setup in **Part 1** specifically for this tutorial.

### Stop the configuration recorder

1. In the AWS Config navigation pane, choose **Settings**.
2. On the **Customer managed recorder** tab, choose **Stop recording**.
3. Choose **Confirm**.

Stopping the recorder stops new resource configuration recording. Previously recorded data and objects already delivered to S3 remain.

### Remove the delivery channel and recorder

The AWS Config Console can stop a customer managed recorder but cannot fully delete its customer managed recorder or delivery channel. Use AWS CloudShell for these two cleanup commands.

1. Choose the **CloudShell** icon in the AWS Console navigation bar.
2. Confirm CloudShell is using the same Region as this tutorial.
3. Run the following command to see the delivery-channel name:

```bash
aws configservice describe-delivery-channels --query "DeliveryChannels[].name" --output table
```

4. If the name is `default`, run:

```bash
aws configservice delete-delivery-channel --delivery-channel-name default
```

5. Run the following command to see the recorder name:

```bash
aws configservice describe-configuration-recorders --query "ConfigurationRecorders[].name" --output table
```

6. If the name is `default`, run:

```bash
aws configservice delete-configuration-recorder --configuration-recorder-name default
```

If either name differs, replace `default` with the exact name shown by its describe command. A delivery channel can be deleted only after the recorder has stopped.

### Delete the S3 delivery bucket

1. Open the **Amazon S3** console.
2. Select the bucket name you recorded in Part 1.
3. Choose **Empty** and follow the confirmation steps.
4. Return to the bucket list, select the same bucket, and choose **Delete**.
5. Enter the bucket name when requested and confirm deletion.

The AWS Config service-linked IAM role may remain in the account. It has no additional IAM charge and can be reused if AWS Config is enabled again.

### Final checkpoint

In the tutorial Region, confirm:

- `tutorial-ebs-encryption-default` is no longer under AWS Config **Rules**.
- The customer managed recorder is no longer listed after the CloudShell deletion.
- The delivery channel describe command returns no tutorial channel.
- The `tutorial-aws-config-...` S3 bucket no longer exists.

## Troubleshooting

### I do not see Get started

AWS Config is probably already set up in this Region. Do not overwrite the existing recording configuration. Add only the tutorial rule and delete only that rule afterward.

### The rule says Insufficient data

Wait for the initial evaluation, refresh, and use **Actions**, **Re-evaluate**. Also confirm the rule is enabled and AWS Config recording is running.

### The delivery channel cannot be deleted

Confirm the customer managed recorder is stopped. Then use `describe-delivery-channels` to verify the exact channel name before rerunning the delete command.

### I cannot delete the S3 bucket

The bucket must be empty first. Delete all current objects and any versions or delete markers if bucket versioning is enabled, then retry the bucket deletion.

## What you learned

- A configuration recorder captures selected AWS resource configurations.
- A delivery channel sends configuration history and snapshots to Amazon S3.
- An AWS Config managed rule evaluates a required configuration without you writing rule code.
- `COMPLIANT` and `NON_COMPLIANT` describe the evaluated configuration; they do not automatically remediate it.
- AWS Config is Regional and charges for configuration items and rule evaluations.
- Deleting a rule, stopping recording, and cleaning up the delivery resources are separate actions.

## Official AWS documentation

- [Manual setup for AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/manual-setup.title.html)
- [Recording resources in the AWS Config Console](https://docs.aws.amazon.com/config/latest/developerguide/select-resources-console.html)
- [Add AWS Config rules](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_add-rules.html)
- [EC2 EBS encryption by default managed rule](https://docs.aws.amazon.com/config/latest/developerguide/ec2-ebs-encryption-by-default.html)
- [Stop the customer managed configuration recorder](https://docs.aws.amazon.com/config/latest/developerguide/managing-recorder_console-stop.html)
- [AWS Config pricing](https://aws.amazon.com/config/pricing/)

Instructions checked on 13 September 2026. AWS Console labels can change slightly over time.
