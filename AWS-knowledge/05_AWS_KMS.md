# 05 — AWS Key Management Service (KMS): very simple tutorial

AWS Key Management Service (**AWS KMS**) creates and controls cryptographic keys used to protect data.

KMS manages the **key**; services such as EBS, S3, and Lambda can use that key to encrypt their data.

## Simple flow

```text
Application or AWS service
        -> asks KMS to use an authorized KMS key
        -> data is encrypted or decrypted
        -> the KMS key material stays protected by KMS
```

Large data is commonly protected with **envelope encryption**:

1. A short-lived data key encrypts the actual data.
2. The KMS key encrypts the data key.
3. Store the encrypted data key beside the encrypted data.
4. An authorized request asks KMS to decrypt the data key when the data is needed.

## Online-shop example

The shop stores customer-order data on an encrypted EBS volume.

1. Choose a suitable KMS key for EBS encryption.
2. Allow the required service and administrators to use the key.
3. Create the encrypted EBS volume.
4. EBS encrypts stored data and its snapshots.
5. If access to the required KMS key is lost, the encrypted storage may become unusable.

## Key choices

- **AWS owned key:** AWS manages it for use across a service; you do not manage its policy.
- **AWS managed key:** visible in your account and managed by an AWS service.
- **Customer managed key:** you control its policy, lifecycle settings, and grants, with added management responsibility and cost.

Use a customer managed key when you need detailed control or audit separation. Do not choose one automatically when the service's default encryption already meets the requirement.

## Security rule

Both IAM permissions and the KMS key policy can affect access. Grant only the required operations. Be especially careful with key disabling and deletion because encrypted data depends on the key.

## What to remember

- KMS protects and controls keys; it is not general-purpose file storage.
- AWS services can integrate with KMS for encryption at rest.
- Envelope encryption uses a data key for data and a KMS key to protect that data key.
- Key access is separate from access to the encrypted resource.
- Losing usable key access can make encrypted data unavailable.

## Practice

1. A user can read an encrypted object but cannot use its customer managed KMS key. Can the read still fail?
2. Why use a data key for a large file?
3. When might a customer managed key be preferable?

<details>
<summary>Suggested answers</summary>

1. Yes. Resource access and key-use permission are separate requirements.
2. Data keys efficiently encrypt the data, while KMS protects the data key.
3. When the organization needs control over key policy, lifecycle, audit separation, or sharing.

</details>

## Official sources

- [AWS KMS cryptography essentials](https://docs.aws.amazon.com/kms/latest/developerguide/kms-cryptography.html)
- [Create a KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html)

Checked 13 September 2026. No AWS resources were created.
