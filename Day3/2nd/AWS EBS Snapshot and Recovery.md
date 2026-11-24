# Project-2: AWS EBS Snapshot and Recovery – Point-in-Time Data Restoration

## 📌 Project Overview

**Task No:** Task-026  
**Session:** 13 – EC2 Configuration & Storage  
**Service:** Amazon EC2 and Amazon EBS  
**Operating System:** Amazon Linux 2023  
**Instance:** `checking`  
**Volume Type:** gp3  
**Snapshot Name:** `Task-026-point-in-time-backup`

### Objective

The objective of this practical is to understand how to create an EBS snapshot, simulate accidental data modification or deletion, and restore the original data using a new EBS volume created from the snapshot.

This practical demonstrates **Point-in-Time Recovery (PITR)** using AWS EBS Snapshots.

---

## 1. Existing EBS Volume Verification

Before creating the snapshot, verify that the additional EBS volume is attached and mounted to the EC2 instance.

Connect to the EC2 instance using MobaXterm.

```bash
lsblk
```

Check the contents of the mounted volume:

```bash
ls -lah /data
```

The additional EBS volume is mounted at:

```text
/data
```

## 2. Create Sample Data

Create a directory for the snapshot recovery demonstration.

```bash
sudo mkdir -p /data/backup-demo
```

Create the first file:

```bash
echo "This is my original data before snapshot" | sudo tee /data/backup-demo/important.txt
```

Create the second file:

```bash
echo "AWS EBS Snapshot Recovery Practical" | sudo tee /data/backup-demo/project.txt
```

Verify the original data:

```bash
cat /data/backup-demo/important.txt
cat /data/backup-demo/project.txt
```

Expected original content:

```text
This is my original data before snapshot
AWS EBS Snapshot Recovery Practical
```

## 3. Create an EBS Snapshot

1. Open the AWS Management Console.
2. Navigate to **EC2 → Elastic Block Store → Volumes**.
3. Select the additional 5 GiB EBS data volume attached to the `checking` instance.
4. Select **Actions → Create snapshot**.
5. Enter the snapshot description:

```text
Task-026-point-in-time-backup
```

6. Click **Create snapshot**.

Navigate to:

**EC2 → Elastic Block Store → Snapshots**

Wait until the snapshot status becomes:

```text
Completed
```

**Important:** Create the snapshot of the additional data volume, not the EC2 root volume.

## 4. Modify and Delete Original Data

After the snapshot is completed, simulate accidental data modification.

Modify the first file:

```bash
echo "Modified data after snapshot" | sudo tee /data/backup-demo/important.txt
```

Delete the second file:

```bash
sudo rm /data/backup-demo/project.txt
```

Verify the modified state:

```bash
ls -l /data/backup-demo/
cat /data/backup-demo/important.txt
```

Expected result:

- `important.txt` contains the modified content.
- `project.txt` is deleted.

This simulates accidental modification and deletion after the backup was created.

## 5. Create a New EBS Volume from the Snapshot

1. Navigate to **EC2 → Elastic Block Store → Snapshots**.
2. Select `Task-026-point-in-time-backup`.
3. Select **Actions → Create volume from snapshot**.
4. Configure the volume:

| Setting | Value |
|---|---|
| Availability Zone | Same AZ as `checking` |
| Volume Type | gp3 |
| Size | 5 GiB or larger if required |

5. Click **Create volume**.

AWS creates a new EBS volume containing the data captured in the snapshot.

## 6. Attach the Restored Volume to EC2

1. Navigate to **EC2 → Elastic Block Store → Volumes**.
2. Select the newly created restored volume.
3. Select **Actions → Attach volume**.
4. Choose the EC2 instance `checking`.
5. Specify a different device name, such as `/dev/sdg`.
6. Click **Attach volume**.

**Note:** On Nitro-based EC2 instances, the Linux device name may appear as `/dev/nvme2n1` instead of the device name specified in the AWS Console.

## 7. Identify the Restored Disk

Connect to the EC2 instance using MobaXterm.

Run:

```bash
lsblk -f
```

Identify the newly attached restored volume by checking its filesystem and device details.

Do not format the restored disk because it already contains the filesystem and data from the snapshot.

## 8. Mount the Restored Volume

Create a separate mount directory:

```bash
sudo mkdir -p /restore
```

Mount the restored volume in read-only mode:

```bash
sudo mount -o ro /dev/nvme2n1 /restore
```

**Important:** Replace `/dev/nvme2n1` with the actual restored disk device identified using `lsblk -f`. If the filesystem appears on a partition, use the correct partition instead.

Read-only mounting helps protect the recovered data during verification.

## 9. Verify Point-in-Time Recovery

Check the restored directory:

```bash
ls -lah /restore/backup-demo/
```

Read the recovered first file:

```bash
cat /restore/backup-demo/important.txt
```

Read the recovered second file:

```bash
cat /restore/backup-demo/project.txt
```

Expected recovered content:

```text
This is my original data before snapshot
AWS EBS Snapshot Recovery Practical
```

### Recovery Verification

| File | Original Volume After Modification | Restored Volume |
|---|---|---|
| important.txt | Modified data after snapshot | Original content recovered |
| project.txt | Deleted | Original file recovered |

**Result:** Both original files were recovered from the EBS snapshot, demonstrating successful point-in-time data restoration.

---

## 10. Important AWS Concepts Learned

### What is an EBS Snapshot?

An EBS Snapshot is a point-in-time backup of an Amazon EBS volume. It can be used to create a new EBS volume containing the data captured at the time of the snapshot.

### What is Point-in-Time Recovery?

Point-in-Time Recovery means restoring data to an earlier state using a backup created before unwanted changes occurred.

### Why Create a New Volume from a Snapshot?

Creating a new volume allows us to recover and inspect the previous data without overwriting the original EBS volume.

### Why Use Read-Only Mounting?

Read-only mounting prevents accidental changes to the recovered data during verification.

### Important Interview Points

1. **What is an EBS Snapshot?**  
   A point-in-time backup of an EBS volume.

2. **Are EBS snapshots incremental?**  
   Yes. EBS snapshots store changed blocks incrementally after the initial snapshot.

3. **Can we create an EBS volume from a snapshot?**  
   Yes. We can create a new EBS volume from an existing snapshot.

4. **Can a snapshot help recover deleted files?**  
   Yes, if those files existed when the snapshot was taken.

5. **Can we attach an EBS volume to multiple EC2 instances?**  
   It depends on the volume type and configuration. Multi-Attach is supported for specific Provisioned IOPS SSD volume types, subject to its limitations.

6. **Why should we select the same Availability Zone when attaching a volume?**  
   An EBS volume must be attached to an EC2 instance in the same Availability Zone.

---

## 11. Problems and Important Notes

- The restored volume may appear under a different Linux device name because EC2 Nitro instances use NVMe device naming.
- Always use `lsblk -f` to identify the correct restored disk.
- Never run `mkfs` on a restored volume because it would overwrite the existing filesystem.
- Mount the restored volume separately to avoid overwriting the original data.
- Snapshot creation is asynchronous. Wait until the snapshot status is `Completed` before using it for the recovery test.
- EBS snapshots are stored independently of the original EBS volume.

## 12. Cleanup and Cost Management

After completing the recovery test and collecting screenshots:

1. Unmount the restored volume:

```bash
sudo umount /restore
```

2. Detach the temporary restored EBS volume through the AWS Console.
3. Delete the temporary restored volume if it is no longer required.
4. Keep the original EBS volume and snapshot if they are needed for future practicals.

**Note:** Do not delete the original `/data` volume or the snapshot if you still need them for upcoming tasks.

---

## Final Outcome

Successfully created an EBS snapshot, simulated accidental data modification and deletion, created a new EBS volume from the snapshot, and verified recovery of the original files.

**Task-026: Completed**

**Key Learning:** AWS EBS Snapshots provide a reliable way to back up EBS volumes and restore data from an earlier point in time.
