---
layout: post
title:  "Shrinking XFS partition"
description: Shrinking XFS partition on linux (Ubuntu).
date:   2017-08-05 23:59:59 +0300
categories: linux
image: /assets/images/common/thumbnails/tux.png
---

XFS is a fairly popular file system on Linux.

Ability to shrink partitions might be useful in cases when you need to shrink one partition
and expand another one.
I had a problem where my root partition was too small, so the only way to expand root partition
was to shrink home and expand root file system.

To manipulate XFS partitions you might need to install **xfsprogs** package. You can grow partition size
using [xfs_growfs](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Storage_Administration_Guide/xfsgrow.html).
But unlike EXT4 file system there is no way to shrink partition. Good luck you're fucked.


## Solution
But there is a solution how to shrink partition. It's not a straightforward solution,
but it saved my hide, when my root partition was way too small.

1. Backup partition image
2. Delete old partition
3. Create new smaller partition (make sure that image still fits partition)
4. Restore image
5. Edit fstab

### Example
For this example, I have created virtual machine with Ubuntu installation.
There is /home (/dev/sda5) and / (/dev/sda1) partitions. In this case we will be shrinking home
partition from ~8.6GB to 4GB.

List of partitions:

    root@ubuntu:/home/ubuntu# fdisk -l
    ...

    Disk /dev/sda: 16 GiB, 17179869184 bytes, 33554432 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x577dacac

    Device     Boot    Start      End  Sectors  Size Id Type
    /dev/sda1  *        2048 15624191 15622144  7.5G 83 Linux
    /dev/sda2       15626238 33552383 17926146  8.6G  5 Extended
    /dev/sda5       15626240 33552383 17926144  8.6G 83 Linux


    Disk /dev/sdb: 4 GiB, 4294967296 bytes, 8388608 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 95A2ABE1-743B-4A77-9133-7EF3B3AF6F5A

    Device     Start     End Sectors Size Type
    /dev/sdb1   2048 8388574 8386527   4G Linux filesystem

As you can see from partition list, there is /dev/sdb1 this partition will be used
to back up our /home partition.

### Preparation
Probably you could shrink home partition using live system.
But if you would like to shrink root partition you could not do it on a live system.

So in this example I will demonstrate a method that should work even with resizing root partition.
For this to work you'll need to boot not from your hardrive, but from Live USB stick or CD.

In this case I have chosen to use **Live Ubuntu CD**.

When you are running from the Live CD open terminal and change to root, because most
of the commands require root access.

    sudo su

### Backing up image
For creating disk images I have chosen to use [**xfsdump**](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Storage_Administration_Guide/xfsbackuprestore.html) application. It allows
to create backup of mounted file system. In most distributions **xfsdump** and
**xfsrestore** are not installed by default.

Install **xfsdump** and **xfsrestore**:
   apt-get install xfsdump

Before making an image of home partition we need to mount it.

    mkdir /mnt/home
    mount /dev/sda5 /mnt/home

Also we need to mount destination partition. This partition should be big enough
to hold backup image.

    mkdir /mnt/external
    mount /dev/sdb1 /mnt/external

Backup image creation:

    xfsdump -l 0 -f /mnt/external/backup /mnt/home

When running xfs dump you must specify backup file name. In this case image name is
**backup**.

After running this command you'll be asked to enter label of backup. In this case
I have used "home" as label.

### Delete old partition
Unmount home partition.

    umount /mnt/home


To delete partition you can use your favorite application. It can be GParted or
other tool.

In this example I have used interactive commandline utility fdisk.

    fdisk /dev/sda
    Welcome to fdisk (util-linux 2.29).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.


    Command (m for help): d
    Partition number (1,2,5, default 5): 5

    Partition 5 has been deleted.

**d** stands for delete. **5th** partition is home partition

Press **w** to write changes to disk.

### Create new smaller partition
To create new partition you can use any tool you like, but for this example I have
chose fdisk.

    fdisk /dev/sda
    Welcome to fdisk (util-linux 2.29).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.

    Command (m for help): n
    All space for primary partitions is in use.
    Adding logical partition 5
    First sector (15628286-33552383, default 15628288): (enter)
    Last sector, +sectors or +size{K,M,G,T,P} (15628288-33552383, default 33552383): +4G

    Created a new partition 5 of type 'Linux' and of size 4 GiB.

**n** command is for creating new partition. **+4G** means that new partition size will be 4GB.

Press **w** to write changes to disk.

Create XFS file system on new partition:

    mkfs.xfs /dev/sda5

Mount partition:

     mount /dev/sda5 /mnt/home

### Restore image
To restore XFS partition you need to know session ID. To find out image session id
run this command:

    xfsrestore -I | grep session

Output of this command:

    session 0:
    		session label:	"home"
    		session id:	7a80f19f-ab34-4598-bf8f-dede406d50dc

So in our case session id is 7a80f19f-ab34-4598-bf8f-dede406d50dc.

Run xfsrestore:

    xfsrestore -f /mnt/external/backup -S 7a80f19f-ab34-4598-bf8f-dede406d50dc /mnt/home

Output:

    dede406d50dc /mnt/home
    xfsrestore: using file dump (drive_simple) strategy
    ...
    xfsrestore: Restore Summary:
    xfsrestore:   stream 0 /mnt/external/backup OK (success)
    xfsrestore: Restore Status: SUCCESS

### Edit fstab
When you are shrinking /home or root partition you need to take into account that
these partitions are mounted via fstab using UUID. New partition will have different
UUID, because of that your system might not boot.

Before rebooting you should mount your root file system and edit fstab file.

Run blkid to find out current UUID of disk.

    blkid
    /dev/sr0: UUID="2017-04-12-03-44-04-00" LABEL="Ubuntu 17.04 amd64" TYPE="iso9660" PTUUID="1b571474" PTTYPE="dos"
    /dev/loop0: TYPE="squashfs"
    /dev/sda1: UUID="3d2b0c24-2aae-4eb6-ae76-b15f3aca32f4" TYPE="xfs" PARTUUID="577dacac-01"
    /dev/sda5: UUID="f359a7c4-bc72-416d-a50a-5869792c2832" TYPE="xfs" PARTUUID="577dacac-05"
    /dev/sdb1: UUID="0882a8db-ae76-44de-9bbe-a453c727ff50" TYPE="xfs" PARTUUID="9f306132-b5fc-4bd5-8a85-a3f9d820276a"

In this case new UUID is 0882a8db-ae76-44de-9bbe-a453c727ff50. You should replace
home partition UUID in fstab with your new partitions UUID.

    mkdir /mnt/root
    mount /dev/sda1 /mnt/root
    vim /mnt/root/etc/fstab


### Summarry
That's it, the solution was a little bit long winded, but at least it works.
If you know a simpler way to resize XFS partition feel free to share it in the
comments section.
