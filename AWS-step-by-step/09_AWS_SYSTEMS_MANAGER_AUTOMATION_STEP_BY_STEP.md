# 09 — AWS Systems Manager Automation step-by-step tutorial

## Goal

Launch one disposable EC2 instance, use AWS Systems Manager Automation runbooks to stop and start it, inspect the execution steps, deliberately run one failed automation, and remove every resource created by the tutorial.

**Difficulty:** Beginner  
**Time:** About 25–30 minutes  
**AWS charges:** The EC2 instance can incur compute charges while running, and its EBS root volume incurs storage charges even while the instance is stopped. Systems Manager Automation can incur charges based on the runbook steps executed. Free Tier coverage depends on your account and total usage. Terminate the instance and verify that its root volume is deleted during cleanup.

## What you will create

| Item | Tutorial value |
| --- | --- |
| EC2 instance name | `tutorial-automation-instance` |
| AMI | Amazon Linux 2023 |
| Instance type | A Free Tier-eligible `t3.micro` or `t2.micro`, when available |
| Security group | `tutorial-automation-sg` with no inbound rules |
| Root EBS volume | Default small `gp3` volume with delete on termination enabled |
| Successful stop runbook | `AWS-StopEC2Instance` |
| Successful start runbook | `AWS-StartEC2Instance` |
| Deliberately failed runbook | `AWS-StopEC2Instance` with an invalid instance ID |

A **runbook** is a Systems Manager document containing an automated sequence of steps. An **Automation execution** is one run of that runbook with specific input values.

The AWS-owned runbooks in this lab call Amazon EC2 APIs directly. The instance does not need an IAM instance profile or Systems Manager Agent registration.

## Part 1 — Launch the disposable EC2 instance

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the upper-right Region selector, choose your normal learning Region. Use the same Region throughout the tutorial.
3. In the top search box, enter **EC2** and open **EC2**.
4. Choose **Instances** in the left navigation pane.
5. Choose **Launch instances**.
6. Under **Name and tags**, enter `tutorial-automation-instance`.
7. Under **Application and OS Images**, select **Amazon Linux** and keep the current **Amazon Linux 2023 AMI**.
8. Under **Instance type**, choose `t3.micro` if the Console marks it as Free Tier eligible. If it is unavailable, choose `t2.micro` when that type is marked eligible.
9. Under **Key pair (login)**, choose **Proceed without a key pair**.

    You will not sign in to this instance. Automation only needs its instance ID.

10. Under **Network settings**, choose **Edit**.
11. Choose the **default VPC** and a default subnet.
12. Set **Auto-assign public IP** to **Disable**.
13. Under **Firewall (security groups)**, choose **Create security group**.
14. For **Security group name**, enter `tutorial-automation-sg`.
15. For **Description**, enter `No inbound access for the Systems Manager Automation tutorial.`
16. Remove every inbound security-group rule. Do not enable SSH, HTTPS, or HTTP access.
17. Under **Configure storage**, keep one small `gp3` root volume.
18. Expand the root-volume details if necessary and confirm **Delete on termination** is **Yes** or selected.
19. Do not add another volume.
20. Under **Advanced details**, keep **IAM instance profile** set to **None**.
21. In **Summary**, confirm that only one instance will launch.
22. Choose **Launch instance**.
23. Choose **View all instances**.

### Record the resource IDs

1. Select `tutorial-automation-instance`.
2. Copy its **Instance ID** into a temporary note. It begins with `i-`.
3. Choose the **Storage** tab.
4. Under **Block devices**, copy the root **Volume ID** into the same note. It begins with `vol-`.

### Checkpoint

Wait until:

- Instance state is **Running**.
- Status checks show **2/2 checks passed**.
- The **Security** tab lists only `tutorial-automation-sg`.
- The root volume shows **Delete on termination: Yes**.

The instance does not require inbound network access for this tutorial.

## Part 2 — Open Systems Manager Automation

1. In the Console search box, enter **Systems Manager** and open **AWS Systems Manager**.
2. In the left navigation pane, choose **Automation**.

    If the item is hidden in the current layout, search the Systems Manager navigation for `Automation`.

3. Choose **Execute automation**.

The runbook list contains documents owned by your account, AWS, and other permitted publishers. In this lab, use only the exact AWS-owned runbook named in each section.

## Part 3 — Stop the instance with a runbook

1. On **Choose document**, search for `AWS-StopEC2Instance`.
2. Select the exact runbook named `AWS-StopEC2Instance` whose owner is **Amazon**.
3. Choose **Next**.
4. For **Execution mode**, choose **Simple execution**.
5. Under **Input parameters**, leave **AutomationAssumeRole** empty.

    When this optional value is empty, Automation uses the permissions of the identity that started the execution.

6. For **InstanceId**, select or paste the exact instance ID recorded in Part 1.
7. Review the input carefully and confirm that it identifies `tutorial-automation-instance`.
8. Choose **Execute**.
9. Wait for **Overall status** to become **Success**.

### Inspect the execution

1. Copy the **Execution ID** into your temporary note.
2. Choose the **Executed steps** or **Execution steps** tab.
3. Open each step and inspect its action, status, start time, and output.
4. Notice that the runbook both requests the state change and waits for the expected EC2 state.
5. Return to the EC2 **Instances** page and refresh it.

### Checkpoint

Confirm both results:

- The Automation execution has status **Success**.
- `tutorial-automation-instance` has state **Stopped**.

Stopping an instance stops compute charges, but its attached EBS volume continues incurring storage charges.

## Part 4 — Start the instance with another runbook

1. Return to Systems Manager **Automation**.
2. Choose **Execute automation**.
3. Search for `AWS-StartEC2Instance`.
4. Select the exact runbook named `AWS-StartEC2Instance` whose owner is **Amazon**.
5. Choose **Next**.
6. For **Execution mode**, choose **Simple execution**.
7. Leave **AutomationAssumeRole** empty.
8. For **InstanceId**, select or paste the exact tutorial instance ID.
9. Keep **FailOnUnexpectedStopped** at its default value of `true`.
10. Choose **Execute**.
11. Wait for **Overall status** to become **Success**.
12. Open the **Executed steps** or **Execution steps** tab and inspect the successful steps.
13. Return to EC2 and refresh the instance list.

### Checkpoint

Confirm both results:

- The start execution has status **Success**.
- `tutorial-automation-instance` has state **Running**.

This proves that Automation can apply and verify an operational change without signing in to the instance.

## Part 5 — Run an intentional failure test

This execution uses a correctly formatted but nonexistent instance ID. It does not target the tutorial instance or any other real instance.

1. Return to Systems Manager **Automation**.
2. Choose **Execute automation**.
3. Select the Amazon-owned `AWS-StopEC2Instance` runbook again.
4. Choose **Next**.
5. Choose **Simple execution**.
6. Leave **AutomationAssumeRole** empty.
7. For **InstanceId**, enter:

```text
i-00000000000000000
```

8. Confirm that this value is all zeroes and is not the real instance ID recorded in Part 1.
9. Choose **Execute**.
10. Wait for the execution to reach a final state.

**Expected result:** Overall status becomes **Failed**. A step reports an error similar to `InvalidInstanceID.NotFound` because that instance does not exist in the selected Region.

### Inspect the failure

1. Open **Executed steps** or **Execution steps**.
2. Select the failed step.
3. Read **Failure details**, **Error message**, or **Output**, depending on the Console layout.
4. Notice which step failed and that later steps did not report success.
5. Return to EC2 and confirm that `tutorial-automation-instance` is still **Running**.

### Checkpoint

You should now have three execution records:

| Runbook | Input | Expected status |
| --- | --- | --- |
| `AWS-StopEC2Instance` | Real tutorial instance ID | Success |
| `AWS-StartEC2Instance` | Real tutorial instance ID | Success |
| `AWS-StopEC2Instance` | `i-00000000000000000` | Failed |

Automation execution history is useful for auditing who ran a runbook, which parameters were used, which steps ran, and why an execution failed.

## Part 6 — Clean up

Terminate only the instance whose **Name** is `tutorial-automation-instance` and whose instance ID matches the value recorded in Part 1. Termination is permanent.

### Terminate the EC2 instance

1. Open the EC2 Console.
2. Choose **Instances**.
3. Select only `tutorial-automation-instance`.
4. Compare its **Instance ID** with the ID recorded in Part 1.
5. Choose **Instance state**, then **Terminate (delete) instance**.
6. Read the confirmation and choose **Terminate (delete)**.
7. Wait until the instance state becomes **Terminated**.

### Verify that the EBS root volume was deleted

1. In the EC2 navigation pane, choose **Volumes** under **Elastic Block Store**.
2. Search for the root volume ID recorded in Part 1.
3. Refresh until that volume disappears.

**Expected result:** The root volume no longer exists because **Delete on termination** was enabled.

If the volume remains in the **Available** state, verify its volume ID carefully, select only that tutorial volume, choose **Actions**, **Delete volume**, and confirm. An available EBS volume can continue incurring storage charges.

### Delete the security group

1. In the EC2 navigation pane, choose **Security Groups** under **Network & Security**.
2. Select only `tutorial-automation-sg`.
3. Choose **Actions**, then **Delete security groups**.
4. Confirm deletion.

If deletion reports that the group is still in use, wait several minutes for the terminated instance's network interface to be removed, then try again.

### Final checkpoint

Confirm that:

- `tutorial-automation-instance` is terminated.
- The recorded root EBS volume no longer exists.
- `tutorial-automation-sg` no longer exists.
- The stop, start, and failed Automation executions remain visible as execution history only.

## Troubleshooting

### The runbook does not appear

Clear other document filters, search for the exact name, and confirm that AWS-owned or **Owned by Amazon** documents are visible. Choose `AWS-StopEC2Instance` or `AWS-StartEC2Instance`, not a similarly named document.

### The execution fails with AccessDenied

Because **AutomationAssumeRole** is empty, the execution uses your signed-in identity. That identity needs permission to start and inspect Systems Manager automations and to describe, stop, and start the selected EC2 instance. Use an authorized learning identity or ask the account administrator to provide the required permissions.

### The successful automation stays In progress

Refresh the execution after a short wait and inspect its current step. Also check the instance in EC2. State transitions such as **Stopping** and **Pending** take time before reaching **Stopped** or **Running**.

### The real instance ID reports NotFound

Confirm that Systems Manager and EC2 are open in the same AWS Region. Copy the ID again from the EC2 instance rather than typing it manually.

### The stop execution succeeds but the instance starts again

Check whether another service, such as an Auto Scaling group, manages the instance. The standalone instance created by this tutorial should not be attached to an Auto Scaling group.

## What you learned

- Systems Manager Automation runs operational workflows defined in runbooks.
- AWS provides managed runbooks for common tasks such as starting and stopping EC2 instances.
- Runbook parameters identify the resource on which the automation acts.
- When `AutomationAssumeRole` is empty, the automation uses the permissions of the identity that starts it.
- Execution steps and outputs show what happened and help diagnose failures.
- A failed execution does not mean the Automation service is broken; its step details explain the cause.
- Stopping an EC2 instance does not remove its EBS storage, while termination can delete the root volume when **Delete on termination** is enabled.

## Official AWS documentation

- [Running a Systems Manager automation](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-working-executing.html)
- [`AWS-StopEC2Instance` runbook](https://docs.aws.amazon.com/systems-manager-automation-runbooks/latest/userguide/automation-aws-stopec2instance.html)
- [`AWS-StartEC2Instance` runbook](https://docs.aws.amazon.com/systems-manager-automation-runbooks/latest/userguide/automation-aws-startec2instance.html)
- [Automation execution statuses](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-statuses.html)
- [Setting up Systems Manager Automation permissions](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-setup.html)
- [EC2 instance lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
- [Systems Manager pricing](https://aws.amazon.com/systems-manager/pricing/)

Instructions checked on 13 September 2026. AWS Console labels and Free Tier offers can change over time.
