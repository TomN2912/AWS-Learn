# 02 — Amazon EBS step-by-step tutorial

## Goal

Create a small Amazon Elastic Block Store (EBS) volume, make a snapshot backup, restore a second volume from that snapshot, and then delete every resource created by the lab.

**Difficulty:** Beginner  
**Time:** About 15–25 minutes, including time for the snapshot to complete  
**AWS charges:** Yes. EBS volumes are charged for provisioned storage while they exist, and snapshots have separate storage charges. Pricing varies by Region. Complete the cleanup section immediately after the lab.

This tutorial does **not** launch an EC2 instance, so you will not mount the volume or write files to it. The aim is to learn the EBS volume and snapshot lifecycle with minimal cost.

## What you will create

| Resource | Name tag |
| --- | --- |
| 1 GiB `gp3` source volume | `tutorial-ebs-source` |
| Snapshot backup | `tutorial-ebs-snapshot` |
| 1 GiB restored volume | `tutorial-ebs-restored` |

An EBS **volume** is a virtual block-storage disk. A **snapshot** is a point-in-time backup that can be used to create another volume.

## Part 1 — Create a small EBS volume

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the top search box, enter **EC2** and open the **EC2** console.
3. In the left navigation pane, find **Elastic Block Store** and choose **Volumes**.
4. Choose **Create volume**.
5. For **Volume type**, select **General Purpose SSD (gp3)**.
6. For **Size**, enter `1` GiB.
7. Leave **IOPS** and **Throughput** at their default values.
8. For **Availability Zone**, choose any standard Availability Zone and write it here: `ap southeast 2a________________`.
9. For **Snapshot ID**, keep **Don't create volume from a snapshot**.
10. Under **Encryption**, enable encryption if it is not already enabled. Keep the account's default EBS KMS key, normally the AWS managed key `aws/ebs`.
11. Under **Tags**, choose **Add tag** and enter:

    - Key: `Name`
    - Value: `tutorial-ebs-source`

12. Choose **Create volume**.
13. Return to **Volumes** and wait until the volume state becomes `Available`.

### Checkpoint

Select the volume and confirm:

- Name is `tutorial-ebs-source`.
- Type is `gp3`.
- Size is `1 GiB`.
- State is `Available`, meaning it is not attached to an EC2 instance.
- Availability Zone matches the one you recorded.
- Encryption is enabled.

The volume is billable even though it is not attached.

## Part 2 — Create a snapshot

1. On the **Volumes** page, select `tutorial-ebs-source`.
2. Choose **Actions**, then **Create snapshot**.
3. For **Description**, enter `Snapshot created for the EBS tutorial.`
4. Under **Tags**, add:

    - Key: `Name`
    - Value: `tutorial-ebs-snapshot`

5. Choose **Create snapshot**.
6. In the left navigation pane, under **Elastic Block Store**, choose **Snapshots**.
7. If you do not see it immediately, refresh the list and filter by `tutorial-ebs-snapshot`.
8. Wait until its status changes from `Pending` to `Completed` before continuing.

### Checkpoint

The completed snapshot is a restore point for the source volume. Snapshot creation is asynchronous, so `Pending` is normal. The snapshot is Regional and can be used to create a volume in an Availability Zone in the same Region.

## Part 3 — Restore a new volume

1. On the **Snapshots** page, select `tutorial-ebs-snapshot`.
2. Choose **Actions**, then **Create volume from snapshot**.
3. Keep **Volume type** as `gp3` and **Size** as `1 GiB`.
4. For **Availability Zone**:

    - Prefer a different Availability Zone from the source volume to demonstrate cross-AZ restore.
    - If your selected Region offers only one usable Availability Zone to your account, use the same one.

5. Do not increase IOPS or throughput.
6. Keep encryption enabled with the inherited or default KMS key.
7. Under **Tags**, add:

    - Key: `Name`
    - Value: `tutorial-ebs-restored`

8. Choose **Create volume**.
9. Open **Volumes** and wait for `tutorial-ebs-restored` to enter the `Available` state.

### Checkpoint

You should now have two unattached volumes:

| Volume | Created from | Expected state |
| --- | --- | --- |
| `tutorial-ebs-source` | Empty volume | `Available` |
| `tutorial-ebs-restored` | Tutorial snapshot | `Available` |

If you chose a different Availability Zone, compare the two volumes' AZ values. This proves that the live volume is an AZ resource, while its Regional snapshot can create a replacement volume in another AZ in that Region.

The restored volume is also billable. Creating a volume from a snapshot does not automatically attach it to an EC2 instance.

## Part 4 — Clean up immediately

Delete the two volumes first, and then delete the snapshot. Confirm the names carefully so you do not delete unrelated resources.

### Delete the restored volume

1. On the **Volumes** page, select only `tutorial-ebs-restored`.
2. Confirm its state is `Available`.
3. Choose **Actions**, then **Delete volume**.
4. Enter `delete` if requested and confirm **Delete**.

### Delete the source volume

1. Select only `tutorial-ebs-source`.
2. Confirm its state is `Available`.
3. Choose **Actions**, then **Delete volume**.
4. Enter `delete` if requested and confirm **Delete**.

### Delete the snapshot

1. In the left navigation pane, choose **Snapshots**.
2. Select only `tutorial-ebs-snapshot`.
3. Choose **Actions**, then **Delete snapshot**.
4. Confirm the deletion.

Deleting the volumes does not automatically delete the snapshot. Deleting the snapshot does not automatically delete volumes that were already restored from it.

### Final checkpoint

Search for each Name tag below and confirm that none remains:

- `tutorial-ebs-source`
- `tutorial-ebs-restored`
- `tutorial-ebs-snapshot`

If your account uses AWS Recycle Bin retention rules, a deleted EBS resource may be retained according to those rules and could remain recoverable or billable during retention.

## Troubleshooting

### The snapshot is still Pending

Snapshot creation is asynchronous. Refresh after a short wait. Do not try to restore it until its status is `Completed`.

### Delete volume is disabled

The volume must be in the `Available` state. A volume in the `In-use` state is attached to an instance and must be safely detached before deletion. The tutorial volumes should never be attached.

### I cannot select a different Availability Zone

The Region or account may not make another AZ available to you. Restore into the same AZ; the snapshot-and-restore lesson still works.

### I cannot enable or change encryption

Your account may enforce EBS encryption by default or restrict which KMS keys you can use. Keep the permitted default encryption setting and ask your account administrator if creation fails.

## What you learned

- An EBS volume is created in one Availability Zone.
- `Available` means the volume exists but is not attached.
- A snapshot is an asynchronous point-in-time backup.
- A completed snapshot can create another volume in the same Region, including in another AZ.
- Unattached volumes and retained snapshots can still create charges.
- Volumes and snapshots have separate lifecycles and must be deleted separately.

## Official AWS documentation

- [Create an Amazon EBS volume](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-creating-volume.html)
- [Create Amazon EBS snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-creating-snapshot.html)
- [Amazon EBS snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html)
- [Delete an Amazon EBS volume](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-deleting-volume.html)
- [Amazon EBS pricing](https://aws.amazon.com/ebs/pricing/)

Instructions checked on 13 September 2026. AWS Console labels can change slightly over time.
