# Week 10: Storage, Backup, and Archiving

## HarborTech Ticket Summary

TKT-2026-0010 focuses on Bright Path Community Services and its mixed storage environment. The client has application disks, shared team files, uploaded documents, and older completed records, but it does not have one consistent backup, recovery, or retention strategy.

The goal is to match each type of data to the correct storage model and explain how it should be protected, retained, and recovered.

## Client Impact

Using the wrong storage type or relying only on primary storage can increase the risk of data loss, slow recovery, and unnecessary cost.

The client needs a clear plan for where active data belongs, how it should be protected, and when older data should move to lower-cost storage.

## AWS Services Involved

### Amazon EBS
EBS provides block storage for EC2 instances. It is a good fit for application disks. Snapshots can provide separate recovery points.

### Amazon EFS
EFS provides shared file storage and is useful when multiple systems need access to the same file system.

### Amazon S3
S3 provides object storage for uploaded documents and other files stored as objects. Versioning can help recover older versions after an overwrite or delete.

### S3 Glacier
Glacier is useful for older records that need long-term retention but are not accessed often.

## Virtualization Connection

AWS hides much of the physical storage hardware, but the storage type still matters. EBS acts like a virtual disk, EFS acts like a shared file system, and S3 stores data as objects.

Even though the storage is virtual, the customer still has to decide how the data is protected, recovered, and retained.

## Evidence Reviewed

The Week 10 ticket shows:

- Application data is stored on an EC2-attached volume.
- Team files are stored separately.
- Uploaded documents are stored as objects.
- Older completed records are still being kept in active storage.
- Some data uses snapshots while other data uses manual copies.
- There is no consistent retention schedule.

## Operational Analysis

The EC2 application disk is best matched with EBS because it needs block storage. The main EBS volume should not be treated as its own backup, so snapshots should be used for recovery points.

Shared team files fit EFS because multiple systems may need access to the same file system.

Uploaded documents fit S3 because they use object storage. Versioning can improve recovery after accidental changes or deletion.

Older completed records should not stay in active storage forever. Lifecycle rules can move them to an archive tier when the retention and retrieval requirements allow it.

## Recommendation

I recommend matching each workload to its access pattern instead of using one storage service for everything.

Use EBS for the application disk and snapshots for recovery. Use EFS for shared files. Use S3 for uploaded documents and versioning when recovery is needed. Use lifecycle rules and Glacier for older records that are rarely accessed.

The client should also create a consistent retention plan instead of relying on manual copies.

## Recovery and Retrieval Verification

For EBS, I would confirm that the snapshot completes successfully and that a new volume could be created from it.

For S3, I would verify that a previous object version can be recovered when versioning is enabled.

For archived records, I would confirm that the lifecycle rule matches the required retention period and that the retrieval process is understood before moving production data.

## Escalation Notes

Production migrations, destructive lifecycle rules, retention-policy changes, or deletion of retained data should require approval.

As an intern, I can review the evidence and recommend changes, but I should not make major production storage or retention changes without authorization.

## Lessons Learned

Week 10 showed me that primary storage, backup, and archive are not the same thing.

The storage choice should start with the access pattern: block, file, object, or archive. After that, the team still needs to decide how the data will be protected and how long it should be kept.

A backup or archive is only useful if the recovery or retrieval process can actually be verified.

## Professional Vocabulary

### Block Storage
Storage presented as a disk or volume to a system.

### File Storage
Storage organized into shared files and folders.

### Object Storage
Storage that keeps data as objects accessed by a key.

### Amazon EBS
Persistent block storage used with EC2.

### Amazon EFS
Managed shared file storage.

### Amazon S3
AWS object storage used for files, versioning, storage classes, and lifecycle management.

### Snapshot
A point-in-time recovery copy of storage such as an EBS volume.

### Versioning
A feature that keeps older versions of an object.

### Lifecycle Rule
A policy that automatically moves or expires objects.

### Archive Storage
Storage for data that is kept long term and accessed less often.

### Retention
The amount of time data must be kept.
