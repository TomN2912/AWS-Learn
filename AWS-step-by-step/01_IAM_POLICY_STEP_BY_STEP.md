# 01 — IAM policy step-by-step tutorial

## Goal

Create a small read-only IAM policy, attach it to an IAM role, and use the IAM Policy Simulator to prove that one action is allowed while another action is denied.

**Difficulty:** Beginner  
**Time:** About 15–20 minutes  
**AWS charges:** IAM does not have an additional charge. This tutorial does not create an S3 bucket or EC2 instance.

## What you will create

| Resource | Name |
| --- | --- |
| Customer managed IAM policy | `tutorial-list-s3-buckets` |
| IAM role | `tutorial-iam-role` |

The role will trust the EC2 service, but you will not attach the role to an EC2 instance. You will not create a password or access key.

## Before you start

- Sign in through your normal AWS access method. Do not use the root user for this lab.
- Your current identity needs permission to create and delete IAM policies and roles, attach and detach policies, and use the IAM Policy Simulator.
- Use a personal sandbox or training account, not a production account.

## Part 1 — Create the policy

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the top search box, enter **IAM** and open **IAM**.
3. In the left navigation pane, choose **Policies**.
4. Choose **Create policy**.
5. In **Policy editor**, choose **JSON**.
6. Remove the example text and paste this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBucketsOnly",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    }
  ]
}
```

7. Choose **Next**. If IAM displays an error, compare your JSON carefully with the example.
8. For **Policy name**, enter `tutorial-list-s3-buckets`.
9. For the optional description, enter `Allows listing S3 bucket names for an IAM tutorial.`
10. Review the permission and choose **Create policy**.

### Checkpoint

You should see a success message and find `tutorial-list-s3-buckets` under **Policies**. The policy allows only the S3 `ListAllMyBuckets` action. It does not allow creating, deleting, or reading objects from buckets.

## Part 2 — Create a role and attach the policy

1. In the IAM left navigation pane, choose **Roles**.
2. Choose **Create role**.
3. For **Trusted entity type**, select **AWS service**.
4. For **Service or use case**, select **EC2**.
5. Choose **Next**.
6. In the permissions search box, enter `tutorial-list-s3-buckets`.
7. Select the checkbox beside your policy and choose **Next**.
8. For **Role name**, enter `tutorial-iam-role`.
9. Review the trusted entity and attached permission, then choose **Create role**.

### Checkpoint

Open `tutorial-iam-role` and check:

- **Trust relationships** shows that the EC2 service can assume the role.
- **Permissions** shows `tutorial-list-s3-buckets`.

These are different jobs: the trust policy says **who can assume the role**, while the permissions policy says **what the role may do**.

## Part 3 — Test the permissions

1. Open the [IAM Policy Simulator](https://policysim.aws.amazon.com/).
https://us-east-1.console.aws.amazon.com/iam/home#/policysim?referrer=https%3A%2F%2Fpolicysim.aws.amazon.com
2. If prompted, sign in using the same AWS account.
3. Use **Principal mode** and select **Roles**.
4. Select `tutorial-iam-role`.
5. Under **Select service**, choose **Amazon S3**.
6. Select the action **ListAllMyBuckets**.
7. Choose **Run simulation**.

**Expected result:** `allowed`. The attached policy explicitly allows this action.

Now test an action that the policy does not grant:

8. Clear the selected action if necessary and select **CreateBucket**.
9. Choose **Run simulation** again.

**Expected result:** `implicitDeny` or denied. No applicable policy allows the action.

The simulator does not make a real S3 request. Its result can differ from a live environment when other controls or resource policies are involved, but it is useful for this basic identity-policy test.

## Part 4 — Clean up

Complete cleanup even though IAM itself has no additional charge. It keeps the account tidy.

### Delete the role

1. Return to the IAM console and choose **Roles**.
2. Select `tutorial-iam-role`.
3. Choose **Delete**.
4. Enter the role name if requested, then confirm **Delete**.

Deleting the role removes its attachment to the policy but does not delete the customer managed policy.

### Delete the policy

1. In the IAM navigation pane, choose **Policies**.
2. Filter for **Customer managed** policies if needed.
3. Select `tutorial-list-s3-buckets`.
4. Choose **Delete**.
5. Enter the policy name if requested, then confirm **Delete**.

### Final checkpoint

Search **Roles** and **Policies**. Neither tutorial resource should remain.

## Troubleshooting

### I cannot create a policy or role

Your signed-in identity probably lacks an IAM permission such as policy or role management. Ask the administrator of your sandbox or training account. Do not work around the restriction with the root user.

### My policy does not appear during role creation

Clear other filters, search for the exact name `tutorial-list-s3-buckets`, and ensure you are viewing customer managed policies.

### I cannot use Principal mode in Policy Simulator

Your identity may lack permission to read IAM principals or call `iam:SimulatePrincipalPolicy`. An administrator can grant the required simulator permissions, or you can review the policy without running this part of the lab.

## What you learned

- An IAM policy is a JSON document that grants or denies actions.
- A policy must be attached to an identity before it grants that identity permissions.
- A role trust policy and a permissions policy answer different questions.
- An action without an applicable `Allow` is implicitly denied.
- Policy Simulator can evaluate permissions without performing the real AWS action.

## Official AWS documentation

- [Create IAM policies in the console](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html)
- [Create an IAM role for an AWS service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html)
- [Test IAM policies with Policy Simulator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)

Instructions checked on 13 September 2026. AWS Console labels can change slightly over time.
