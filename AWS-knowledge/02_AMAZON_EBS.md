# 02 — Amazon Elastic Block Store (EBS)

## Start here — EBS in 5 minutes

Amazon **Elastic Block Store (EBS)** is a persistent virtual disk for an Amazon EC2 server.

```text
EC2 server -> attached EBS volume -> files and database data
                                      |
                                      +-> snapshot backup
```

Simple setup:

1. Create an EBS volume in the same Availability Zone as the EC2 instance.
2. Attach it to the instance.
3. In the operating system, format a new empty volume and mount it.
4. Save application files or database data on it.
5. Create snapshots for backup and recovery.

Stopping the EC2 instance does not erase the EBS data, but deletion and the volume's termination setting still matter. A live EBS volume belongs to one Availability Zone; a snapshot can be used to create a replacement volume in another Availability Zone in the same Region.

**Simple choice:** start by considering `gp3` for a normal application disk. Use S3 for objects, EFS for a shared file system, and instance store only for temporary data that can be recreated.

Amazon EBS provides persistent block storage for Amazon EC2 virtual servers. Use it when a server needs a disk for its operating system, application files, or database data, and that data must survive the server being stopped.

The most important boundary: **an EBS volume belongs to one Availability Zone. Persistent storage does not automatically provide recovery from a whole Availability Zone outage.**

## 1. Foundations: what block storage means

Amazon EC2 supplies compute: a virtual server that runs an operating system and applications. EBS supplies storage that the operating system sees as a block device. A block device supports reading and writing portions of data at particular positions. A filesystem organizes those blocks into files and directories; a database can update its data through the filesystem.

| Term | Meaning |
| --- | --- |
| Volume | An individual EBS storage device with a configured capacity and performance |
| Root volume | The disk containing the operating system used to boot the instance |
| Data volume | An additional disk used for application or database data |
| Region | An AWS geographical area, such as Sydney |
| Availability Zone (AZ) | An isolated infrastructure location within a Region |
| Snapshot | A point-in-time backup used to create a new volume |
| Mount | Make a filesystem accessible at a directory or drive location in the operating system |
| GiB / MiB | Binary units of capacity; throughput is often measured in MiB per second |

EBS is network-attached storage, separate from the EC2 host’s local disks. The application normally performs operating-system file operations, rather than calling an EBS API for each read or write. You can attach multiple volumes to one instance, but a volume and its attached instance must be in the same AZ. [AWS volume overview](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes.html)

## 2. Running example: a payroll processing server

Consider an illustrative business application that imports employee timesheets, validates them, and writes a local processing database. It runs on one EC2 instance. The business requires the database to survive maintenance stops and instance replacement, and accepts recovery from a recent backup after an AZ outage.

```text
AWS Region
|
+-- Availability Zone A
|   +-- EC2 payroll application
|       +-- Root EBS volume: operating system
|       +-- Encrypted EBS data volume: processing database
|
+-- Regional EBS snapshot backups
    +-- Restore a NEW volume in AZ B when recovery is needed
        +-- Attach to a replacement EC2 instance in AZ B
```

The diagram represents attachments and a backup/restore path. It does not represent continuous replication between the live volumes in different AZs.

### Follow one data update

1. A timesheet reaches the payroll application.
2. The application validates it and updates its database.
3. The database writes through the operating system to its mounted EBS data volume.
4. The database uses its normal transaction and flush mechanisms to ensure committed data is written appropriately; a value held only in application memory is not yet protected by disk persistence.
5. A later backup captures the volume’s stored state. A completed backup can be used to recreate storage after a failure.

EBS supplies storage, not database transaction management. AWS manages the storage infrastructure; the customer manages the instance’s operating system, filesystem, application, access, capacity planning, and recovery procedures.

### Follow the volume lifecycle

Create a volume in the instance’s AZ, choose its type and encryption, and attach it to the instance. For a new empty data volume, create a filesystem and mount it. A restored volume already containing a filesystem should be mounted appropriately, not formatted again. Configure persistent mounting if the application needs the disk after reboot.

When replacing a server in the same AZ, stop application writes, unmount and detach the retained data volume cleanly, then attach and mount it on the replacement server. Ensure the replacement application can interpret the existing data. [AWS volume lifecycle](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-lifecycle.html)

## 3. Persistence, termination, and failure boundaries

| Event | Storage consequence |
| --- | --- |
| Reboot the instance | EBS data remains |
| Stop and later start an EBS-backed instance | EBS data remains; storage charges continue |
| Terminate the instance | Each volume’s DeleteOnTermination setting determines whether it is deleted |
| Delete a volume | Recover using an available backup; persistence does not prevent deletion |
| Lose access to an entire AZ | The same live volume cannot be attached to an instance in another AZ |

Root volumes are normally configured for deletion on termination, while additional data volumes are commonly retained. Check the actual setting rather than assuming a default. Set the payroll data volume to be retained if instance replacement must preserve it.

EBS replicates volume data within its AZ to protect against individual infrastructure component failures. This is not a backup against application corruption, accidental deletion, or an AZ-wide outage. The payroll design therefore needs snapshots and a tested recovery process. [AWS EBS features](https://docs.aws.amazon.com/ebs/latest/userguide/EBSFeatures.html), [AWS EBS documentation overview](https://aws.amazon.com/documentation-overview/ebs/)

## 4. Choosing a volume type

**SSD** means solid-state drive; these volumes suit frequent small reads and writes. **HDD** means hard disk drive; these types suit larger sequential transfers.

| Type | Practical fit | Choice to think through |
| --- | --- | --- |
| gp3 | General application disks, boot volumes, many ordinary databases | Start here for the illustrative payroll workload, then measure |
| gp2 | Existing general-purpose SSD workloads | Performance is tied more closely to size and may involve burst credits |
| io2 Block Express | Demanding transactional systems with stringent latency and sustained I/O needs | Justify the additional cost with measured requirements |
| io1 | Provisioned IOPS workloads, often existing designs | Compare current requirements with io2 |
| st1 | Frequently accessed, large sequential datasets | Throughput-oriented HDD; unsuitable as a boot volume |
| sc1 | Infrequently accessed, sequential datasets that still need an attached disk | Lower-cost HDD with performance tradeoffs; unsuitable as a boot volume |

The older magnetic `standard` type also exists; focus on current types when learning new designs. “Database” alone does not require io2: a modest database may be well served by gp3. [AWS volume types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)

## 5. Understanding performance

**IOPS** means input/output operations per second: how many individual reads or writes can be performed. **Throughput** measures how much data moves per second. **Latency** measures how long an individual operation takes.

For an illustrative workload, 1,000 operations per second at 16 KiB each transfers approximately 15.6 MiB/s. The same operation count with much larger requests demands more throughput. This is why a database making small random updates and a reporting job scanning large files can need different storage configurations.

Performance is constrained by the volume, the instance’s EBS capabilities, the operating system, and application behavior. Increasing volume performance will not solve an instance bandwidth bottleneck or a slow database query.

Use Amazon CloudWatch, AWS’s monitoring service, to inspect volume activity, queueing, and available performance/status metrics. Also inspect operating-system disk usage and application latency. Filesystem free space generally needs monitoring inside the guest, for example through the CloudWatch agent; it is not the same as provisioned EBS capacity. [AWS I/O characteristics and monitoring](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-io-characteristics.html)

### Growing the payroll volume

With supported Elastic Volumes configurations, you can increase capacity and change performance or type while a volume is in use. For gp3, capacity and performance can be configured separately within supported limits.

After increasing the EBS volume size, extend the partition if necessary and then the filesystem so the operating system can use the extra space. EBS does not automatically perform every guest operating-system step. You cannot directly shrink an EBS volume; migrate data to a smaller volume if needed. Take a suitable backup before storage changes. [AWS Elastic Volumes](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-modify-volume.html)

## 6. Snapshots and recovery

EBS snapshots are incremental backups: after the initial backup, subsequent snapshots store changed blocks. Each retained snapshot is a usable restore point; you do not manually assemble a chain to restore it.

Regional snapshots are stored in AWS-managed S3 storage and replicated across AZs. They do not appear as ordinary files in your S3 buckets. Create a new volume from a snapshot in the required AZ, then attach it to an instance there. For recovery in another Region, arrange a snapshot copy to that Region.

Backups require configuration: schedule them using Amazon Data Lifecycle Manager or AWS Backup. For payroll, choose frequency based on acceptable lost processing work, not simply “daily backups” by habit. [AWS snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html)

### Consistency matters

A snapshot captures data written to the volume, not application buffers still in memory. Coordinate database backups, flushes, or a pause in writes when application-consistent recovery is required. If data spans multiple disks, coordinate those disks; multi-volume snapshots help capture a crash-consistent set, while application consistency still requires application-aware handling. [AWS snapshot creation](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-creating-snapshot.html)

Deleting an earlier snapshot does not break a later retained snapshot: AWS preserves blocks needed by retained backups. Deleting a snapshot also does not delete an already-restored volume. [AWS snapshot deletion](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-deleting-snapshot.html)

**Recovery point objective (RPO)** is the acceptable amount of lost recent data. **Recovery time objective (RTO)** is the acceptable time to restore service. Hourly completed backups might support roughly an hour of data loss under suitable timing and consistency assumptions, but do not guarantee a short RTO. Provisioning compute, restoring storage, starting the database, and validating payroll all take time.

Volumes restored from snapshots can require initialization as blocks are fetched. Fast Snapshot Restore or a provisioned initialization rate can help with predictable readiness, at additional cost. Measure the actual recovery procedure. [AWS volume initialization](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-initialize.html)

## 7. Security: connect this to IAM

Use [your IAM lesson](01_AWS_IAM.md) to separate management permissions from application file access. IAM policies control AWS actions such as creating volumes, attaching them, creating snapshots, and deleting resources. Once a disk is attached and mounted, operating-system and database permissions govern ordinary application reads and writes. A process does not need an IAM permission for each filesystem operation.

EBS encryption uses AWS Key Management Service (KMS) keys. It protects encrypted volume data at rest, traffic between the instance and volume, and associated snapshots. The customer must retain appropriate key access: losing access to a required KMS key can prevent use or recovery of encrypted storage.

Encryption by default is a per-Region account setting for new EBS storage; enabling it does not retroactively encrypt existing unencrypted volumes. An existing unencrypted volume requires an encrypted replacement workflow, such as using a snapshot to create an encrypted volume. [AWS EBS encryption](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-encryption.html)

For payroll, combine encrypted storage with restricted snapshot access, limited deletion permissions, and appropriate operating-system accounts. Disk encryption does not prevent an authorized application from reading data it is permitted to access.

## 8. EBS and its closest alternatives

| Requirement | Suitable starting point | Reason |
| --- | --- | --- |
| Persistent disk for an EC2 operating system or local database | EBS | Block storage for the server |
| Temporary processing data that can be recreated | EC2 instance store | Host-local storage; do not rely on it surviving stop, termination, or host loss |
| Shared Linux files accessed by servers across AZs | Amazon EFS | Managed shared filesystem access |
| Store completed reports or uploads as objects through an API | Amazon S3 | Object storage rather than an attached disk |

### Does Multi-Attach make EBS a shared filesystem?

No. Supported io1/io2 configurations can attach a volume to multiple compatible instances in the same AZ. The application or cluster-aware filesystem must coordinate access and writes. Mounting a normal filesystem read/write on multiple machines can corrupt data. Multi-Attach neither supplies cross-AZ storage nor replaces EFS for ordinary shared directories. [AWS Multi-Attach](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html)

## 9. Two more realistic uses

### Business reporting: sequential scans

A reporting server reads large local transaction extracts in sequence, calculates totals, and publishes a finished report. An st1 data volume can suit frequent large sequential scans when its measured throughput meets the job deadline. The server still needs a supported root volume type.

The tradeoff is poorer suitability for small random reads and dependence on volume performance characteristics. If the requirement changes to keeping finished reports for occasional retrieval without an always-attached disk, store those outputs in S3 instead.

### Development environment: retain work through shutdowns

A developer builds software on EC2 and saves source checkouts, dependencies, and local databases on gp3. At the end of the day the instance stops; the next start uses the retained data. Snapshots provide restore points before risky environment changes.

The tradeoff is continuing storage cost while compute is stopped, and backups becoming stale between runs. If multiple developers in different AZs must share the same live Linux directory, evaluate EFS instead of trying to attach one gp3 volume to all their servers.

## 10. Cost and limits

You mainly pay for provisioned volume capacity and, depending on type and settings, provisioned performance. For example, an illustrative 200 GiB volume holding 30 GiB of files is charged for its provisioned 200 GiB capacity, not just those files.

Retained or unattached volumes still cost money. Snapshots have separate storage charges, and faster recovery features or cross-Region copying can add charges. Review unused volumes, snapshot retention, and excess performance settings. Check current Region-specific rates rather than memorizing a price. [AWS EBS pricing](https://aws.amazon.com/ebs/pricing/)

Account quotas, such as total storage by volume type, may be adjustable. The same-AZ attachment requirement is an architectural boundary, not a quota you can increase. Volume size/performance ceilings and instance capabilities must both fit the design.

## 11. Common exam traps

- **“Persistent means never deleted.”** Termination settings and explicit deletion still matter.
- **“EBS replication protects against an AZ outage.”** Live volume replication stays within its AZ; plan backup recovery or application replication across AZs.
- **“A snapshot provides instant failover.”** It provides a restore point, not a running replacement application.
- **“Any database needs io2.”** Choose based on latency, sustained I/O, durability needs, and cost.
- **“Bigger storage fixes every slow disk.”** Check IOPS, throughput, instance limits, and application behavior separately.
- **“Multi-Attach means shared storage across AZs.”** Its AZ and coordination constraints still apply.

## What to remember

- EBS is persistent block storage for EC2, scoped to one AZ.
- Check DeleteOnTermination for every important volume.
- Start with workload requirements; gp3 is a useful general-purpose candidate.
- Snapshots protect restore points, while application consistency and recovery testing remain your responsibility.
- Separate disk capacity, IOPS, throughput, and latency when diagnosing performance.

## Practice — design payroll storage

Assume one EC2 payroll server in AZ A needs 200 GiB of persistent database storage. Its measured workload fits gp3. It must survive instance stops and replacement, use encrypted storage, and recover in AZ B after an AZ outage. The business accepts losing up to one hour of recent work; the recovery-time target still needs agreement. These are illustrative requirements, not a production sizing assessment.

1. Describe where the database data goes during a write and what configuration keeps its volume after instance termination.
2. Justify gp3 instead of st1 for frequent small database updates. What evidence might make you reconsider gp3 for io2?
3. AZ A becomes unavailable. Explain the recovery steps and what determines how much recent work is lost.
4. The volume grows from 200 GiB to 300 GiB, but the filesystem still reports its old size. What is missing?
5. The requirement changes: two servers in different AZs need the same shared Linux files. Is attaching this EBS volume to both a solution? Distinguish shared files from a database requiring high availability.

### Suggested answers — try the prompts first

1. The database writes through the operating system to the mounted encrypted EBS data volume. Set DeleteOnTermination to false for that volume, and arrange snapshots separately. Retention preserves storage when replacing compute; it does not protect against every failure.
2. gp3 suits small transactional I/O; st1 favors sequential throughput. Reconsider io2 if measured sustained performance, latency, or durability requirements justify it, after checking the EC2 instance can support the required performance.
3. Use the latest suitable completed snapshot to create an encrypted volume in AZ B, provide authorized KMS access, attach it to replacement compute there, mount it, and recover and validate the database. Lost work depends on the actual recoverable snapshot point and backup consistency. Schedule and monitor backups frequently enough to meet the one-hour target, allowing for failures and timing. Agree and test an RTO separately.
4. Extend the partition if needed and the filesystem inside the operating system. Increasing the EBS block device alone does not complete the guest-side change.
5. No: the volume cannot attach across AZs. EFS is a candidate for shared Linux files. For a highly available database, use a supported database replication/failover architecture or evaluate a managed database deployment; a shared directory alone does not provide database availability or safe concurrent writes.

---

Official AWS references linked throughout were checked on 9 September 2026. This is a conceptual lesson; no AWS resources were provisioned.
