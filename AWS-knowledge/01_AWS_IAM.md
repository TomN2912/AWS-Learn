# 01 — AWS Identity and Access Management (IAM)

## Start here — IAM policy in 5 minutes

An **IAM policy** is a JSON permission document. It answers: **who may do what, to which AWS resource, and under what conditions?**

This policy allows its attached role to read one file from Amazon S3:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::example-shop-orders/daily.csv"
  }]
}
```

- `Effect`: allow or deny.
- `Action`: the permitted AWS operation.
- `Resource`: the object the action can affect.
- `Condition`: an optional extra rule, such as requiring a particular network.

Attach the policy to an identity such as a role. AWS starts with **deny**, permits a request only when an applicable policy allows it, and an applicable explicit `Deny` overrides an `Allow`.

**Simple rule:** grant only the actions and resources that are required. This is called **least privilege**.

IAM controls who can access AWS resources and what actions they can perform. Start by identifying the caller, the requested action, and the resource; then determine which policies apply.

## Core terms

**Authentication** verifies who is making a request. **Authorization** determines whether that identity may perform the requested action. Signing in successfully does not mean every action is permitted.

| Term | Meaning | Typical use |
| --- | --- | --- |
| Principal | An identity making an AWS request, such as a user or an assumed role session | Identify who is requesting access |
| IAM user | An identity in an AWS account that can have long-term credentials | Cases that specifically require an IAM user |
| IAM group | A collection of IAM users that receive permissions together | Apply a policy to multiple users; groups cannot be assumed |
| IAM role | An identity that trusted callers can assume to receive temporary credentials | Applications, federated people, and cross-account access |
| Policy | A JSON document describing permissions or access constraints | Allow reading selected S3 objects |
| AWS STS | AWS Security Token Service, which issues temporary security credentials | Obtain credentials when assuming a role |
| ARN | Amazon Resource Name, an identifier for an AWS resource | Identify the objects covered by a policy |
| Federation | Using an external identity provider to access AWS | Employees use their organization’s sign-in |

An IAM user belongs to an account; it is not itself an AWS account. IAM identities are global within the account rather than recreated separately in every Region.

## Choosing an identity

For employees, prefer federation and temporary credentials. IAM Identity Center provides centrally managed access to AWS accounts and applications. For applications running on AWS, use IAM roles instead of putting access keys in code.

The account root user has special account-level authority. Protect it with multi-factor authentication (MFA), which requires an additional authentication factor, and reserve it for tasks that require root. Apply **least privilege**: grant only the actions and resources needed for a task.

## Roles have two separate permission questions

1. **Who can assume the role?** The role’s trust policy identifies trusted principals and any conditions.
2. **What can the role do?** Permissions policies attached to the role authorize actions after it is assumed.

For example, trusting the EC2 service does not automatically grant access to S3. The role also needs appropriate S3 permissions. For cross-account role assumption, configure trust in the destination account and permission for the caller to assume the role in the source account.

## Reading a policy

An identity-based policy attaches to a user, group, or role. A resource-based policy attaches to a resource, such as an S3 bucket, and can identify the principal receiving access.

| Element | Question it answers |
| --- | --- |
| Effect | Is this statement an Allow or a Deny? |
| Action | Which API operations does it cover? |
| Resource | Which resources does it cover? |
| Condition | Under what additional circumstances does it apply? |
| Principal | Who does a resource-based policy or trust policy apply to? |

A normal identity-based permissions policy does not include Principal because its attachment identifies the identity.

This illustrative policy allows a role to read objects under one bucket prefix:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadMonthlyReports",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-company-reports/monthly/*"
    }
  ]
}
```

Here, `monthly/` is an object-key prefix, and `*` matches objects below it. The bucket name is illustrative. `Version` is the policy language version, not the date the policy was written.

This statement does not grant uploading, deleting, or listing objects. `s3:ListBucket` applies to a bucket ARN; `s3:GetObject` applies to object ARNs. An application that already knows an object key can request it without listing the bucket.

## How AWS decides whether to allow a request

For ordinary permission evaluation, start with these rules:

- Access is denied by default: this is an **implicit deny**.
- A relevant Allow can authorize access, subject to other applicable controls.
- An applicable **explicit Deny** overrides an Allow.

Do not assume that one attached Allow guarantees access. A permissions boundary limits permissions available to a user or role, and an AWS Organizations service control policy (SCP) constrains permissions for affected accounts. Neither grants permission by itself. Session policies can further restrict a role session.

Resource-based permissions introduce additional rules, especially when they name a user, role, or role session directly. The exact evaluation depends on the principal and account relationship; consult the evaluation reference for these cases rather than assuming every policy is simply combined the same way.

## Small architecture example

An application on Amazon EC2, AWS’s virtual-server service, needs to download monthly reports from S3, AWS’s object storage service.

```text
EC2 application
    -> uses temporary credentials from its attached IAM role
    -> requests GetObject for monthly/report.csv
    -> AWS evaluates applicable policies
    -> S3 returns the object when authorized
```

An **instance profile** is the container used to attach an IAM role to EC2. The AWS SDK can obtain and refresh the supplied role credentials automatically. The application does not need stored IAM user access keys.

## Common mistakes

- Treating a trust policy as an S3 permissions policy: trust permits role assumption; separate permissions authorize resource access.
- Giving a role AdministratorAccess for a single read operation: scope the action and resource to the actual requirement.
- Assuming MFA grants permissions: it strengthens authentication; authorization still depends on policies.
- Assuming a missing Allow and an explicit Deny behave identically: another applicable Allow can resolve an implicit deny, but cannot override an explicit Deny.
- Using a bucket ARN for GetObject: object actions require object ARNs.
- Assuming IAM approval guarantees a successful request: networking and other service requirements still apply. For example, objects encrypted with a customer managed KMS key can require additional key permissions.

## Sources

Official AWS references checked on 5 September 2026:

- [IAM FAQs](https://aws.amazon.com/iam/faqs/)
- [IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [How to use trust policies with IAM roles](https://aws.amazon.com/blogs/security/how-to-use-trust-policies-with-iam-roles/)
- [IAM policy evaluation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic_policy-eval-denyallow.html)

## What to remember

- Authentication identifies the caller; authorization checks the requested operation.
- Prefer temporary credentials: federation for people and roles for applications.
- A role’s trust policy and permissions policies answer different questions.
- Grant only required actions on required resources.
- An applicable explicit Deny wins; boundaries and SCPs limit permissions without granting them.

## Practice — EC2 reads monthly reports

**Requirements:** An EC2 application must read objects under `monthly/` in `example-company-reports`. It must not upload or delete objects, and it must not store long-term access keys. It already knows the object keys. Assume the bucket is in the same account, there are no additional permission grants or applicable denies, and no customer managed KMS key permissions are needed.

**Solution:** Attach an EC2-trusted role through an instance profile and give it the example GetObject policy. The application uses temporary role credentials. The action and object prefix match the reading requirement; no upload or delete permissions are granted.

Answer these prompts before expanding the answers:

1. The application requests `annual/report.csv`. Will the example policy authorize it? Why?
2. The application now needs to discover available monthly reports. Which additional action and resource type are needed?
3. A bucket policy explicitly denies the role’s GetObject request. Does the role’s Allow win?
4. The role trusts EC2 but has no S3 permissions. Can the application read the reports under the stated assumptions?
5. Someone proposes storing an IAM user access key on the instance. Which part of the existing solution already meets that need?

<details>
<summary>Suggested answers</summary>

1. No. The object is outside the allowed `monthly/` prefix.
2. Add `s3:ListBucket` on `arn:aws:s3:::example-company-reports`, using an `s3:prefix` condition to constrain permitted listing requests to the required monthly prefix. Keep the object-level GetObject permission separately.
3. No. The applicable explicit Deny overrides the Allow.
4. No. Trust enables role assumption but does not authorize reading S3 objects.
5. The attached role provides temporary credentials that the SDK can obtain and refresh; a stored long-term key is unnecessary.

</details>
