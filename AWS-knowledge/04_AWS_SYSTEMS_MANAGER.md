# 04 — AWS Systems Manager: very simple tutorial

AWS Systems Manager helps administrators view, access, patch, configure, and operate servers from a central AWS service.

A server configured for Systems Manager is called a **managed node**. It needs:

- Systems Manager Agent (**SSM Agent**);
- permission to communicate with Systems Manager; and
- network connectivity to the required service endpoints.

## Four useful tools

| Tool | Simple purpose |
| --- | --- |
| Session Manager | Open a controlled shell session without opening inbound SSH or RDP ports |
| Run Command | Run a command on one or many managed nodes |
| Patch Manager | Apply and report operating-system patches |
| Parameter Store | Store configuration values; `SecureString` can protect sensitive values with KMS |

## Online-shop example

Several EC2 web servers need a configuration check.

```text
Administrator -> Systems Manager Run Command
              -> selected managed EC2 nodes
              -> command output and status
```

1. EC2 instances run SSM Agent and use an IAM role with the required Systems Manager permissions.
2. The administrator selects the nodes by ID or tags.
3. Run Command sends an approved Systems Manager document.
4. Each available node runs the command and reports its result.

Do not put passwords directly in commands. Store sensitive values in a properly restricted secret store or encrypted `SecureString` parameter.

## When to use it

- Manage many servers consistently.
- Connect without maintaining a bastion host or opening inbound management ports.
- Automate repetitive operational work.

Systems Manager is not a replacement for IAM. IAM still decides who may start sessions, run commands, or read parameters.

## What to remember

- A managed node needs agent, permission, and connectivity.
- Session Manager is for interactive access.
- Run Command is for remote commands at scale.
- Patch Manager handles patch operations.
- Parameter Store holds configuration values.

## Practice

1. A node has SSM Agent but no suitable IAM role. Will it become a working managed node?
2. Which tool fits an interactive troubleshooting session?
3. Why is Run Command better than manually signing in to 50 servers for the same task?

<details>
<summary>Suggested answers</summary>

1. No. It also needs authorization and service connectivity.
2. Session Manager.
3. It applies a controlled command consistently at scale and returns execution status.

</details>

## Official sources

- [What is AWS Systems Manager?](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)
- [Running commands on managed nodes](https://docs.aws.amazon.com/systems-manager/latest/userguide/running-commands.html)

Checked 13 September 2026. No AWS resources were created.
