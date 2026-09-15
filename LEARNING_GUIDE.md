# AWS learning guide

## Topic lessons

1. [AWS Identity and Access Management (IAM)](AWS-knowledge/01_AWS_IAM.md) — identities, roles, policies, permission evaluation, and an EC2-to-S3 practice scenario.

2. [Amazon Elastic Block Store (EBS)](AWS-knowledge/02_AMAZON_EBS.md) — persistent disks, volume types, snapshots, encryption, recovery, and payroll storage practice.

3. [AWS Config](AWS-knowledge/03_AWS_CONFIG.md) — configuration history, compliance rules, and a simple encrypted-EBS example.

4. [AWS Systems Manager](AWS-knowledge/04_AWS_SYSTEMS_MANAGER.md) — managed nodes, remote sessions, commands, patching, and parameters.

5. [AWS Key Management Service (KMS)](AWS-knowledge/05_AWS_KMS.md) — managed encryption keys, envelope encryption, access, and service integration.

6. [AWS Lambda](AWS-knowledge/06_AWS_LAMBDA.md) — event-driven code, triggers, execution roles, and retry-safe design.

7. [Amazon EventBridge](AWS-knowledge/07_AMAZON_EVENTBRIDGE.md) — events, buses, rules, targets, and an order-processing example.

Read the lesson, explain its key ideas in your own words, and answer the practice prompts before viewing the suggested answers.

## Step-by-step tutorials

1. [IAM policy hands-on tutorial](AWS-step-by-step/01_IAM_POLICY_STEP_BY_STEP.md) — create a read-only policy and role, test allowed and denied actions, and clean up.

2. [Amazon EBS hands-on tutorial](AWS-step-by-step/02_EBS_STEP_BY_STEP.md) — create a small encrypted volume, snapshot it, restore another volume, and delete all billable resources.

3. [AWS Config hands-on tutorial](AWS-step-by-step/03_AWS_CONFIG_STEP_BY_STEP.md) — configure narrow recording, evaluate EBS encryption by default, view compliance, and clean up.

4. [AWS Systems Manager hands-on tutorial](AWS-step-by-step/04_AWS_SYSTEMS_MANAGER_STEP_BY_STEP.md) — create, retrieve, update, version, and delete a Parameter Store value.

5. [AWS KMS hands-on tutorial](AWS-step-by-step/05_AWS_KMS_STEP_BY_STEP.md) — create a symmetric key, encrypt and decrypt text, test a disabled key, and schedule deletion.

6. [AWS Lambda hands-on tutorial](AWS-step-by-step/06_AWS_LAMBDA_STEP_BY_STEP.md) — create and deploy a Python function, test successful and failed events, inspect its logs, and clean up.

7. [Amazon EventBridge hands-on tutorial](AWS-step-by-step/07_AMAZON_EVENTBRIDGE_STEP_BY_STEP.md) — route a matching custom order event to Lambda, prove a non-matching event is ignored, and clean up.

8. [Amazon RDS for PostgreSQL hands-on tutorial](AWS-step-by-step/08_AMAZON_RDS_POSTGRESQL_STEP_BY_STEP.md) — create a PostgreSQL DB instance, connect with CloudShell, run SQL success and failure tests, prove persistence, and clean up.

9. [AWS Systems Manager Automation hands-on tutorial](AWS-step-by-step/09_AWS_SYSTEMS_MANAGER_AUTOMATION_STEP_BY_STEP.md) — run managed stop and start workflows against EC2, inspect successful and failed executions, and clean up.

## Exam practice

- [Exam 1](exam-questions/exam-1.md) — existing questions and answer explanations.
