# Project-1: AWS EBS Volume Management – Attach, Format, Mount and Configure Persistent Storage on EC2

## 1. Project Overview

**Session:** 13 – EC2 Configuration & Storage  
**Task:** Task-025  
**Project:** Attach and mount an additional EBS volume to an EC2 instance, format it with ext4, and configure persistent mounting.

### Objective

The objective of this practical was to understand AWS Elastic Block Store (EBS) and learn how to attach additional storage to an EC2 instance.

Created an additional EBS volume, attached it to a Linux EC2 instance, formatted it using the ext4 filesystem, mounted it at `/data`, and configured persistent mounting using `/etc/fstab`.

Finally, rebooted the EC2 instance and verified that the additional volume was automatically mounted.

## 2. AWS Services and Technologies Used

- Amazon EC2
- Amazon Elastic Block Store (EBS)
- Amazon Linux 2023
- MobaXterm
- Linux Disk Management
- ext4 Filesystem
- `/etc/fstab`
- UUID-based Mounting

## 3. Architecture

```text
                  AWS Cloud
                      |
                      v
                 EC2 Instance
                Amazon Linux 2023
                      |
             --------------------
             |                  |
             v                  v
       Root EBS Volume    Additional EBS
             |                  |
             v                  v
         /dev/nvme0n1      /dev/nvme1n1
             |                  |
             v                  v
         Root Filesystem    ext4 Filesystem
             |                  |
             v                  v
             /                 /data
                                |
                                v
                         Persistent Mounting
                            /etc/fstab
```

## 4. Implementation Steps

### Step 1: Select EC2 Instance

1. Opened AWS Management Console.
2. Navigated to EC2 → Instances.
3. Selected the existing EC2 instance named `checking`.
4. Verified that the instance was in Running state.
5. Checked its Availability Zone.

The EBS volume was created in the same Availability Zone as the EC2 instance.

### Step 2: Create Additional EBS Volume

Navigated to:

EC2 → Elastic Block Store → Volumes → Create volume.

Configured the following:

| Configuration | Value |
|---|---|
| Volume Type | gp3 |
| Volume Size | 5 GiB |
| Availability Zone | Same as EC2 instance |
| Encryption | Default |

Created the additional EBS volume.

### Step 3: Attach EBS Volume to EC2

1. Selected the newly created EBS volume.
2. Opened Actions → Attach volume.
3. Selected the EC2 instance named `checking`.
4. Specified the device name presented by AWS.
5. Attached the volume successfully.

### Step 4: Identify the Additional Disk

Connected to the EC2 instance using MobaXterm.

Executed:

```bash
lsblk
```

This command displays available block devices and their mount points.

Identified the additional disk:

`/dev/nvme1n1`

The original root disk was:

`/dev/nvme0n1`

Checked filesystem information:

```bash
lsblk -f
```

Also checked mounted filesystems:

```bash
df -h
```

Verified that the additional disk was separate from the root filesystem.

### Step 5: Format the Additional Disk with ext4

Formatted the newly attached EBS volume:

```bash
sudo mkfs -t ext4 /dev/nvme1n1
```

**Explanation:**

- `sudo` – Execute with administrator privileges.
- `mkfs` – Create a filesystem.
- `-t ext4` – Specify the ext4 filesystem.
- `/dev/nvme1n1` – Additional EBS device.

Verified the filesystem:

```bash
lsblk -f
```

Confirmed that the additional volume had the ext4 filesystem.

### Step 6: Create Mount Point

Created a directory named `/data`:

```bash
sudo mkdir -p /data
```

This directory acts as the mount point for the additional EBS volume.

### Step 7: Mount the EBS Volume

Mounted the additional disk:

```bash
sudo mount /dev/nvme1n1 /data
```

Verified the mount:

```bash
df -h
```

Observed the additional volume mounted at `/data`.

Example output:

```text
Filesystem       Size  Used Avail Use% Mounted on
/dev/nvme0n1p1   8.0G  1.7G  6.3G  21% /
/dev/nvme1n1     4.9G   24K  4.6G   1% /data
```

### Step 8: Configure Persistent Mounting

To ensure that the EBS volume automatically mounts after an EC2 reboot, configured `/etc/fstab`.

First, identified the filesystem UUID:

```bash
sudo blkid
```

Created a backup of the existing fstab file:

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Opened the configuration file:

```bash
sudo vi /etc/fstab
```

Added the following entry using the actual UUID returned by `blkid`:

```text
UUID=<actual-volume-UUID> /data ext4 defaults,nofail 0 2
```

**Explanation:**

| Field | Meaning |
|---|---|
| UUID | Unique filesystem identifier |
| /data | Mount point |
| ext4 | Filesystem type |
| defaults,nofail | Default options; do not block boot if volume is unavailable |
| 0 | Dump backup setting |
| 2 | Filesystem check order |

Used UUID instead of the device name to identify the filesystem reliably.

### Step 9: Test Persistent Mount Configuration

Unmounted the volume:

```bash
sudo umount /data
```

Tested the `/etc/fstab` configuration:

```bash
sudo mount -a
```

Verified the mounted filesystem:

```bash
df -h
```

Confirmed that `/data` was mounted successfully.

### Step 10: Reboot and Verify Persistent Mount

Rebooted the EC2 instance:

```bash
sudo reboot
```

Reconnected to the server using MobaXterm.

Executed:

```bash
df -h
```

Observed the additional EBS volume still mounted at `/data`.

Final verification:

```text
/dev/nvme1n1  4.9G  24K  4.6G  1%  /data
```

**Result:** Persistent mounting successfully verified after reboot.

## 5. Problems Faced and Troubleshooting

### Problem 1: Identifying the correct EBS device

**Issue:**

After attaching the additional EBS volume, the Linux device name was not necessarily the same as the device name specified in the AWS Console.

**Investigation:**

Used:

```bash
lsblk
lsblk -f
```

**Resolution:**

Identified the additional disk as `/dev/nvme1n1` and distinguished it from the existing root disk `/dev/nvme0n1`.

**Learning:**

AWS NVMe-based EC2 instances can expose EBS devices with NVMe names. Always verify the actual Linux device before formatting.

### Problem 2: Persistent mounting after reboot

**Issue:**

A normal `mount` command does not automatically guarantee that a volume will be mounted after a system reboot.

**Investigation:**

Reviewed the purpose of `/etc/fstab` and filesystem UUIDs.

**Resolution:**

Added the additional EBS volume's UUID to `/etc/fstab`, tested the configuration using:

```bash
sudo mount -a
```

Then rebooted the EC2 instance and verified the mount using:

```bash
df -h
```

**Result:**

The `/data` mount remained available after reboot.

**Learning:**

Use UUID-based entries in `/etc/fstab` for persistent filesystem mounting.

## 6. Final Result

| Activity | Result |
|---|---|
| Create additional EBS volume | Successful |
| Attach volume to EC2 | Successful |
| Identify additional disk | Successful |
| Format using ext4 | Successful |
| Create `/data` mount point | Successful |
| Mount EBS volume | Successful |
| Configure `/etc/fstab` | Successful |
| Test using `mount -a` | Successful |
| Reboot EC2 instance | Successful |
| Verify persistent mounting | Successful |

## 7. Important Linux Commands Learned

| Command | Purpose |
|---|---|
| `lsblk` | List block devices |
| `lsblk -f` | Display filesystem information |
| `df -h` | Display mounted filesystem usage |
| `sudo blkid` | Display filesystem UUIDs |
| `sudo mkfs -t ext4` | Format a disk using ext4 |
| `sudo mkdir -p /data` | Create a mount point |
| `sudo mount` | Mount a filesystem |
| `sudo umount /data` | Unmount a filesystem |
| `sudo vi /etc/fstab` | Edit persistent mount configuration |
| `sudo mount -a` | Mount filesystems configured in fstab |
| `sudo cp` | Create a backup copy |
| `sudo reboot` | Reboot the Linux server |

## 8. Key Learnings

1. Understood AWS EBS and its purpose.
2. Learned how to create and attach additional EBS volumes.
3. Understood the relationship between EC2 Availability Zones and EBS volumes.
4. Learned how to identify disks using `lsblk`.
5. Understood Linux filesystems and ext4 formatting.
6. Learned how to create mount points and mount additional storage.
7. Understood filesystem UUIDs.
8. Learned how to configure persistent mounting using `/etc/fstab`.
9. Tested mount configuration using `mount -a`.
10. Verified persistent mounting after reboot.

## 9. Conclusion

Successfully attached an additional 5 GiB EBS volume to an Amazon Linux EC2 instance.

Formatted the volume with ext4, mounted it at `/data`, and configured UUID-based persistent mounting through `/etc/fstab`.

Finally, rebooted the EC2 instance and confirmed that the additional EBS volume was automatically mounted.

This practical provided hands-on experience with AWS block storage, Linux disk management, filesystem formatting, mounting, and persistent storage configuration.
