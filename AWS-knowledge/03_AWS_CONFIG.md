# 03 — AWS Config: very simple tutorial

AWS Config records how supported AWS resources are configured and can check whether those configurations follow your rules.

It answers two useful questions:

- **What did this resource look like before?**
- **Does it meet our required configuration now?**

## Simple flow

```text
AWS resource changes
        -> AWS Config records a configuration item
        -> a Config rule evaluates the resource
        -> result: COMPLIANT or NON_COMPLIANT
```

A **configuration item** is a snapshot of a resource's settings and relationships. A **Config rule** describes the required state. AWS supplies managed rules, and you can create custom rules when needed.

## Online-shop example

The shop requires every EBS volume to be encrypted.

1. Enable AWS Config in the Region and select the resource types to record.
2. Add an AWS Config managed rule that checks EBS encryption.
3. Someone creates an unencrypted EBS volume.
4. Config records the volume and evaluates it.
5. The volume appears as `NON_COMPLIANT`.
6. The team investigates or configures an approved remediation action.

AWS Config detects and evaluates configuration. It does not automatically block the original creation. Use preventive controls, such as IAM or organization policies, when a bad configuration must be denied before it exists.

## When to use it

- Audit resource configuration changes.
- Find resources that do not follow security rules.
- Investigate when a setting changed.

Do not confuse it with Amazon CloudWatch, which focuses on operational metrics, logs, and alarms. AWS Config focuses on resource configuration and compliance.

## What to remember

- Recorder = captures supported resource configuration.
- Rule = evaluates the desired configuration.
- Result = compliant or noncompliant.
- It is primarily detective unless remediation is configured.
- Configure it for the accounts and Regions you need to observe.

## Practice

1. A security group is changed to allow traffic from anywhere. What can Config help you discover?
2. Will a Config rule always prevent that change?
3. Why must you enable recording in the required Regions?

<details>
<summary>Suggested answers</summary>

1. It can record the changed configuration, show history, and evaluate it against an applicable rule.
2. No. Evaluation is generally detective; prevention needs a suitable preventive control.
3. Config records resources according to its configured account and Regional scope.

</details>

## Official source

- [How AWS Config works](https://docs.aws.amazon.com/config/latest/developerguide/how-does-config-work.html)

Checked 13 September 2026. No AWS resources were created.
