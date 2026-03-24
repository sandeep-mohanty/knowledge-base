# Fix LVM Issues on Linux + Logical Volume Guide with Commands
**A Practical Guide to Linux LVM (Logical Volume Management)**

![](https://miro.medium.com/v2/format:webp/0*PL-yGIh_HgQpiJng.png)

LVM provides more **flexible, scalable, and safer storage management** compared to traditional partitioning. It is a powerful tool that simplifies disk management, reduces downtime, and adapts to growing system needs — making it essential knowledge for Linux professionals.

---

### What Problems Does LVM Solve?
Traditional disk partitioning is rigid. Once partitions are created, resizing or managing them later often requires downtime, reformatting, or data loss. **LVM solves these problems by providing flexibility and dynamic storage management.**

**LVM helps you:**
* **Resize disks and file systems** without downtime
* **Combine multiple physical disks** into a single logical storage pool
* **Allocate storage on demand** as data grows
* **Easily manage storage** for changing application needs
* **Take snapshots** for backups and testing
* **Reduce operational complexity** in production environments

---

### What Is LVM?
**Logical Volume Management (LVM)** is a Linux storage management layer that sits between physical disks and file systems. Instead of working directly with fixed disk partitions, LVM lets you create flexible logical volumes that can be resized, moved, and managed easily.



**LVM Components:**
* **Physical Volume (PV):** Actual disks or disk partitions
* **Volume Group (VG):** A pool of storage created from one or more PVs
* **Logical Volume (LV):** Virtual partitions created from a VG and used by the OS

---

### Common LVM Use Cases

1.  **Dynamic Disk Expansion:** Increase disk space for `/var`, `/home`, or `/data` without stopping applications.
    * *Example:* A database runs out of space → add a new disk → extend the LV → resize filesystem → done.
2.  **Combining Multiple Disks:** Merge multiple smaller disks into a single large logical volume.
    * *Example:* 2 × 500GB disks → 1TB logical volume for application data.
3.  **Snapshots for Backup & Testing:** Create point-in-time snapshots for backups or safe testing.
    * *Example:* Take a snapshot before system upgrades or patching.
4.  **Better Storage Utilization:** Allocate space only when needed instead of over-provisioning.
    * *Example:* Start with a small LV and expand as usage grows.
5.  **Disaster Recovery & Migration:** Move logical volumes between disks with minimal disruption.
    * *Example:* Migrate data from old storage to new disks online.
6.  **Separation of Data & OS:** Isolate application data from the operating system.
    * *Example:* Keep `/var/lib/mysql` or `/opt/app` on separate logical volumes.

---

### Real command examples

**1. Check currently mounted filesystems:**
```bash
[root@Tusharjahdav ~]# df –h
Filesystem Size Used Avail Use% Mounted on
/dev/sda2   18G 2.8G   14G  17% /
tmpfs      497M 228K  497M   1% /dev/shm
/dev/sda1  291M  34M  242M  13% /boot
```

**2. Check physical disks detected by the OS:** (Assuming you have added new HDDs via VMware settings and ran `echo "- - -" > /sys/class/scsi_host/host0/scan` or rebooted)
```bash
[root@Tusharjahdav ~]# fdisk –l
# (Existing OS Drive)
Disk /dev/sda: 21.5 GB...
# (New 2GB drive added)
Disk /dev/sdb: 2147 MB...
# (Another New 2GB drive added - to be used later)
Disk /dev/sdc: 2147 MB...
```

---

## Phase 1: Creating LVM from Scratch
Taking a new raw disk (`/dev/sdb`), turning it into an LVM Physical Volume, creating a Volume Group, and then a Logical Volume.

### Step 1: Prepare the Physical Partition
Create a partition on `/dev/sdb` and set its type to LVM (Hex code **8e**).

```bash
[root@Tusharjahdav ~]# fdisk /dev/sdb
Command (m for help): n   (Create new partition)
p                         (Primary)
1                         (Partition number 1)
<Enter>                   (Default start cylinder)
<Enter>                   (Default end cylinder - use whole disk)
Command (m for help): t   (Change partition type)
Selected partition 1
Hex code (type L to list codes): 8e  (This is the code for Linux LVM)
Changed system type of partition 1 to 8e (Linux LVM)
Command (m for help): w   (Write changes and exit)
```
Sync the kernel partition table:
```bash
[root@Tusharjahdav ~]# partprobe -s
# OR
[root@Tusharjahdav ~]# sync
```

### Step 2: Create the Physical Volume (PV)
Initialize the partition for use by LVM.
```bash
[root@Tusharjahdav ~]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
# Verify:
[root@Tusharjahdav ~]# pvdisplay
```

### Step 3: Create the Volume Group (VG)
Create a pool of storage called **myvol** using the physical volume created above.
```bash
[root@Tusharjahdav ~]# vgcreate myvol /dev/sdb1
  Volume group "myvol" successfully created
# Verify:
[root@Tusharjahdav ~]# vgdisplay
# Note Free PE / Size. (e.g., 2.00 GB total)
```

### Step 4: Create the Logical Volume (LV)
Carve out a 500MB chunk from the **myvol** group named **lv1**.
```bash
[root@Tusharjahdav ~]# lvcreate -L 500M -n lv1 myvol
  Logical volume "lv1" created
# Verify:
[root@Tusharjahdav ~]# lvdisplay
# Note LV Path is usually /dev/myvol/lv1 or /dev/mapper/myvol-lv1
```

---

## Phase 2: Formatting and Mounting
The Logical Volume is now a usable block device. It requires a filesystem and mounting.

### Step 1: Create the Filesystem (Format)
This example uses **ext3** (modern systems might use **ext4** or **xfs**).
```bash
[root@Tusharjahdav ~]# mkfs.ext3 /dev/myvol/lv1
```

### Step 2: Mount the LV
Create a directory and mount the volume temporarily.
```bash
[root@Tusharjahdav ~]# mkdir /Dell
[root@Tusharjahdav ~]# mount /dev/myvol/lv1 /Dell
# Check if mounted:
[root@Tusharjahdav ~]# df -h
/dev/mapper/myvol-lv1  485M  11M  449M   3% /Dell
```

### Step 3: Permanent Mounting (/etc/fstab)
To ensure it mounts automatically on reboot, edit `/etc/fstab`.
```bash
[root@Tusharjahdav ~]# vi /etc/fstab
# Add the following line at the end:
/dev/myvol/lv1    /Dell    ext3    defaults    0 0
```

---

## Phase 3: Managing LVM (The Real Power)

### Scenario A: Extending a Logical Volume (Online)
With free space in the Volume Group **myvol**, **lv1** can be increased while mounted and online.

**1. Extend the LVM Boundary:**
* Add a specific amount: `lvextend -L +200M /dev/myvol/lv1`
* Set a specific total size: `lvextend -L 1200M /dev/myvol/lv1`
* Use remaining free space:
```bash
[root@Tusharjahdav ~]# lvextend -l +100%FREE /dev/myvol/lv1   
Extending logical volume lv1 to 1.99 GiB   
Logical volume lv1 successfully resized
```

**2. Resize the Filesystem:**
The LVM "container" is bigger, but the filesystem inside (ext3) must be updated.
```bash
[root@Tusharjahdav ~]# resize2fs /dev/myvol/lv1
# Note: If using XFS filesystem, use xfs_growfs instead.
# Verify new size:
[root@Tusharjahdav ~]# df -h
```

### Scenario B: Extending the Volume Group with a New Disk
If the Volume Group is full, add a new hard drive (`/dev/sdc`) to the pool.

1.  **Prepare the new disk (Partition as 8e):** Repeat `fdisk` steps for `/dev/sdc`.
2.  **Create Physical Volume:**
    ```bash
    [root@Tusharjahdav ~]# pvcreate /dev/sdc1
    ```
3.  **Extend the Volume Group:** Add the new PV to the existing VG.
    ```bash
    [root@Tusharjahdav ~]# vgextend myvol /dev/sdc1
      Volume group "myvol" successfully extended
    # Verify the VG size (should be ~4GB):
    [root@Tusharjahdav ~]# vgdisplay
    ```

### Scenario C: Reducing a Logical Volume (Offline — Risky!)
> ⚠️ **WARNING:** Reducing is dangerous. Always backup data first. This cannot be done online.

**1. Unmount the filesystem:**
```bash
[root@Tusharjahdav ~]# umount /Dell
```
**2. Force check the filesystem (Mandatory):**
```bash
[root@Tusharjahdav ~]# e2fsck -f /dev/myvol/lv1
```
**3. Shrink the filesystem FIRST:**
```bash
[root@Tusharjahdav ~]# resize2fs /dev/myvol/lv1 1000M
```
**4. Reduce the LVM partition SECOND:**
```bash
[root@Tusharjahdav ~]# lvreduce -L 1000M /dev/myvol/lv1
WARNING: Reducing active logical volume to 1000.00 MiB
THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce lv1? [y/n]: y
```
**5. Mount and verify:**
```bash
[root@Tusharjahdav ~]# mount -a
[root@Tusharjahdav ~]# df -h
```

---

## Phase 4: Removing LVM Components
To completely remove LVM configuration, reverse the creation order: **Unmount -> LV Remove -> VG Remove -> PV Remove.**

1.  **Unmount and clean fstab:**
    ```bash
    umount /Dell 
    # (Edit /etc/fstab and remove the /Dell line)
    ```
2.  **Remove Logical Volume(s):**
    ```bash
    lvremove /dev/myvol/lv1
    ```
3.  **Remove Volume Group:**
    ```bash
    vgremove myvol
    ```
4.  **Remove Physical Volume(s):**
    ```bash
    pvremove /dev/sdb1 pvremove /dev/sdc1
    ```

---

### Bonus: Managing Swap
Extending swap using a regular partition (not LVM).

**1. Check current swap:**
```bash
[root@centoshost ~]# swapon -s
Filename      Type       Size      Used   Priority
/dev/sda2     partition  3833852   0      -1
```
**2. Prepare partition for swap:** Use `fdisk /dev/sdc` and set partition type to **82** (Linux Swap).
**3. Format as swap:**
```bash
mkswap /dev/sdc1
```
**4. Turn on new swap space:**
```bash
swapon /dev/sdc1
# Verify:
free -m
```
**5. Make permanent:** Get the UUID and add to `/etc/fstab`.
```bash
blkid /dev/sdc1
vi /etc/fstab
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  swap  swap  defaults  0 0
```

---

### LVM backup and Configuration
LVM automatically backs up volume group metadata whenever a change is made.

* **Backup Metadata manually:** `vgcfgbackup myvol`
* **Restore Metadata:** `vgcfgrestore myvol` (Useful if a PV was accidentally wiped).
* **Configuration File (`/etc/lvm/lvm.conf`):** Exclude devices (like CD-ROMs) from LVM scans.
    ```bash
    vi /etc/lvm/lvm.conf
    # Search for 'filter' section to exclude cdrom:
    filter = [ "r|/dev/cdrom|", "a/.*/" ]
    ```
    