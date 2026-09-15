# 05 — AWS Key Management Service step-by-step tutorial

## Goal

Create a symmetric customer managed AWS KMS key, encrypt and decrypt sample text, prove that a disabled key cannot decrypt it, re-enable the key, and schedule it for deletion.

**Difficulty:** Beginner  
**Time:** About 20–25 minutes  
**AWS charges:** Customer managed KMS keys and cryptographic API requests can incur charges. Key storage is prorated while the key is active. AWS does not charge for a customer managed key while it is scheduled for deletion, unless deletion is later cancelled.

## What you will create

| Item | Tutorial value |
| --- | --- |
| Key type | Symmetric |
| Key usage | Encrypt and decrypt |
| Regionality | Single-Region key |
| Alias entered in the Console | `tutorial-kms-key` |
| Alias used by the AWS CLI | `alias/tutorial-kms-key` |
| Sample plaintext file | `kms-plaintext.txt` |
| Sample ciphertext file | `kms-ciphertext.bin` |

A KMS key is the protected cryptographic key managed by AWS KMS. An alias is a friendly name that lets you refer to the key without copying its generated key ID.

## Part 1 — Create the KMS key

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the upper-right Region selector, choose your normal learning Region.
3. In the top search box, enter **KMS** and open **Key Management Service**.
4. In the left navigation pane, choose **Customer managed keys**.
5. Choose **Create key**.
6. For **Key type**, select **Symmetric**.
7. For **Key usage**, select **Encrypt and decrypt**.
8. Under **Advanced options**:

    - Keep **Key material origin** as **KMS**.
    - Keep **Regionality** as **Single-Region key**.

9. Choose **Next**.
10. For **Alias**, enter `tutorial-kms-key`.

    Do not enter the `alias/` prefix in the Console. AWS adds it automatically.

11. For **Description**, enter `Symmetric key created for the AWS KMS tutorial.`
12. Add this tag:

    - Key: `Purpose`
    - Value: `Tutorial`

13. Choose **Next**.
14. On **Define key administrative permissions**, select the IAM role you are currently using for this tutorial.
15. Keep **Allow key administrators to delete this key** selected so that you can complete the cleanup section.
16. Choose **Next**.
17. On **Define key usage permissions**, select the same IAM role. This allows that role to call cryptographic operations such as `Encrypt` and `Decrypt`.
18. Choose **Next**.
19. Review the key configuration and key policy.
20. Choose **Finish**.

### Checkpoint

On **Customer managed keys**, find `tutorial-kms-key` and confirm:

- Status is `Enabled`.
- Key type is `Symmetric`.
- Key usage is `Encrypt and decrypt`.
- Regionality is `Single-Region`.
- The complete alias is `alias/tutorial-kms-key`.

The key policy identifies who can administer and use the key. Creating a key does not automatically give every identity permission to use it.

## Part 2 — Create and encrypt sample text

1. Choose the **CloudShell** icon in the AWS Console navigation bar.
2. Confirm CloudShell is using the same Region as the KMS key.
3. Create a small plaintext file:

```bash
printf '%s' 'Hello from the KMS tutorial' > kms-plaintext.txt
```

4. Display the file:

```bash
cat kms-plaintext.txt
```

**Expected result:**

```text
Hello from the KMS tutorial
```

5. Encrypt the file with your KMS key:

```bash
aws kms encrypt \
  --key-id alias/tutorial-kms-key \
  --plaintext fileb://kms-plaintext.txt \
  --output text \
  --query CiphertextBlob | base64 --decode > kms-ciphertext.bin
```


This command encrypts the contents of kms-plaintext.txt using your AWS KMS key and saves the encrypted binary data in kms-ciphertext.bin.

Line by line:
- aws kms encrypt
  Calls the AWS KMS Encrypt operation.
- \
  Continues the command on the next line. It makes a long command easier to read.
- --key-id alias/tutorial-kms-key
  Selects the KMS key whose alias is tutorial-kms-key.
- --plaintext fileb://kms-plaintext.txt
  Reads the data to encrypt from kms-plaintext.txt.
  fileb:// tells the AWS CLI to read the file as raw binary data.
- --output text
  Returns the result as plain text instead of a JSON document.
- --query CiphertextBlob
  Extracts only the encrypted value named CiphertextBlob from the AWS response.
- | base64 --decode
  Sends the Base64-encoded ciphertext to the base64 program and converts it into its original binary representation.
- > kms-ciphertext.bin
  Writes that binary ciphertext into kms-ciphertext.bin. If the file already exists, it is overwritten.
The complete data flow is:
kms-plaintext.txt
        ↓
AWS KMS encrypts the contents
        ↓
KMS returns Base64-encoded ciphertext
        ↓
base64 --decode converts it to binary
        ↓
kms-ciphertext.bin
The output file will not be readable as normal text. To recover the original content, you must use the KMS Decrypt operation with access to the same KMS key.

6. Confirm the ciphertext file was created:

```bash
ls -l kms-ciphertext.bin
```

Encrypted data—ciphertext—is binary: a sequence of bytes that may contain characters unsafe for JSON or terminals.
Base64 converts those binary bytes into ordinary text characters so AWS can safely return them in a JSON response.
Plaintext
   ↓ KMS encryption
Binary ciphertext
   ↓ Base64 encoding
Base64 text
A Base64 value might look like:
AQICAHj7H3kM8xP9...
Why decode it back to binary?
The command saves the actual ciphertext to a binary file:
base64 --decode > kms-ciphertext.bin
Later, KMS can read that binary file directly:
aws kms decrypt \
  --ciphertext-blob fileb://kms-ciphertext.bin
The transformations are:
KMS returns Base64 text
        ↓ base64 --decode
Binary ciphertext file
        ↓ KMS decrypt
Original plaintext
Important distinction:
- Base64 encoding changes binary data into transport-safe text. It provides no security.
- Base64 decoding changes that text back into binary. It does not decrypt anything.
- KMS decryption uses the KMS key to recover the original plaintext.



### Checkpoint

`kms-ciphertext.bin` should exist and have a nonzero size. It contains encrypted binary data, not readable plaintext.

The `fileb://` prefix tells the AWS CLI to read the plaintext as binary data. The KMS `Encrypt` response is Base64-encoded, so the command decodes it before saving the binary ciphertext.

## Part 3 — Decrypt the ciphertext

1. Run:

```bash
aws kms decrypt \
  --ciphertext-blob fileb://kms-ciphertext.bin \
  --output text \
  --query Plaintext | base64 --decode
```
aws kms decrypt \
  --ciphertext-blob fileb://kms-ciphertext.bin \
  --output text \
  --query Plaintext | base64 --decode > decrypted-file.txt

aws kms decrypt \
  --ciphertext-blob fileb://kms-ciphertext.bin \




**Expected result:**

```text
Hello from the KMS tutorial
```

For ciphertext created by a symmetric KMS key, the encrypted blob contains metadata that lets KMS determine which key was used. The caller must still be authorized to use that key.

## Part 4 — Disable the key and test again

1. Return to the AWS KMS Console.
2. Choose **Customer managed keys**.
3. Select the checkbox beside `tutorial-kms-key`.
4. Choose **Key actions**, then **Disable**.
5. Read the warning and confirm the disable action.
6. Wait until the key status shows `Disabled`.
7. Return to CloudShell and run the decrypt command again:

```bash
aws kms decrypt \
  --ciphertext-blob fileb://kms-ciphertext.bin \
  --output text \
  --query Plaintext | base64 --decode
```

**Expected result:** The command fails because a disabled KMS key cannot be used for cryptographic operations. The error can appear as `DisabledException` or another message stating that the key is not in a valid state.

If the command succeeds immediately after disabling, wait briefly and retry because key-state changes can be eventually consistent.

### Re-enable and verify

1. In **Customer managed keys**, select `tutorial-kms-key`.
2. Choose **Key actions**, then **Enable**.
3. Wait until the status returns to `Enabled`.
4. Run the decrypt command again.

**Expected result:** `Hello from the KMS tutorial` appears again.

This proves that possession of ciphertext is not enough. The key must be enabled and the caller must have permission to use it.

## Part 5 — Clean up

### Remove the CloudShell files

Run:

```bash
rm -f kms-plaintext.txt kms-ciphertext.bin
```

rm -f *



Confirm that no tutorial files remain:

```bash
ls -l kms-plaintext.txt kms-ciphertext.bin
```

**Expected result:** CloudShell reports that the files do not exist.

### Schedule the KMS key for deletion

AWS KMS does not allow immediate key deletion. Customer managed keys require a waiting period of 7–30 days.

1. Return to AWS KMS and choose **Customer managed keys**.
2. Select only `tutorial-kms-key`.
3. Choose **Key actions**, then **Schedule key deletion**.
4. Read the deletion warning carefully.
5. For **Waiting period**, enter `7` days.
6. Select the confirmation checkbox.
7. Choose **Schedule deletion**.

### Final checkpoint

The key status should be `Pending deletion`, and the key details should show its scheduled deletion date.

While pending deletion:

- The key cannot encrypt or decrypt data.
- You can cancel deletion before the waiting period ends.
- After the waiting period, deletion is permanent and ciphertext that depends on the key cannot be decrypted.
- The alias is deleted when the KMS key is permanently deleted.

Do not cancel deletion for this tutorial. If you cancel it, the key must be enabled again before it can be used, and key charges apply as though deletion had never been scheduled.

## Troubleshooting

### AccessDeniedException appears during encrypt or decrypt

Open the key and inspect the **Key policy**. Confirm the role used by CloudShell is included as a key user and that its IAM permissions do not block the required KMS operation.

### NotFoundException appears for the alias

Confirm that the key and CloudShell are in the same Region. In CLI commands, the alias must include the `alias/` prefix: `alias/tutorial-kms-key`.

### The Console created alias/alias/tutorial-kms-key

The `alias/` prefix was entered into the Console alias field. The Console adds that prefix automatically. Use the alias actually displayed by KMS in your commands, or create a correctly named alias from the key's **Aliases** tab.

### I cannot schedule deletion

The selected key administrator may not have permission to call `kms:ScheduleKeyDeletion`, or the key policy may prevent deletion. Review the key policy and the **Allow key administrators to delete this key** choice made during creation.

## What you learned

- A symmetric KMS key can encrypt and decrypt data.
- An alias is a friendly, Regional identifier for a key.
- The key policy and IAM permissions control administration and cryptographic use.
- Disabling a key prevents new cryptographic operations without immediately deleting it.
- KMS ciphertext is unusable when the required key is unavailable.
- Customer managed KMS keys require a 7–30 day deletion waiting period.

## Official AWS documentation

- [Create a symmetric KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/create-symmetric-cmk.html)
- [AWS CLI encrypt command](https://docs.aws.amazon.com/cli/latest/reference/kms/encrypt.html)
- [Enable and disable KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/enabling-keys.html)
- [Schedule KMS key deletion](https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys-scheduling-key-deletion.html)
- [AWS KMS pricing](https://aws.amazon.com/kms/pricing/)

Instructions checked on 13 September 2026. AWS Console labels can change slightly over time.
