# Disk Management and Logical Volume Setup

This document outlines the steps to manage Logical Volume Management (LVM) on a Red Hat system.



## Step 1: Creating Physical Volumes

1. **List available disks:**
   ```bash
   lsblk
   ```

2. **Create physical volumes on `/dev/vdb`, `/dev/vdc`, and `/dev/vdd`:**
   ```bash
   pvcreate /dev/vdb
   pvcreate /dev/vdc
   pvcreate /dev/vdd
   ```
3. **Create a volume group named `vg_lab` using the physical volumes:**
   ```bash
   vgcreate vg_lab /dev/vdb /dev/vdc /dev/vdd
   ```
## Step 2: Displaying Volume Group Information

1. **Display information about the volume group `vg_lab`:**
   ```bash
   vgdisplay vg_lab
   ```



## Step 3: Creating Logical Volumes and Filesystems

1. **Create a logical volume named `lv_data` with a size of 4GB:**
   ```bash
   lvcreate -L 4G -n lv_data vg_lab
   ```

2. **Format the logical volume with the `ext4` filesystem:**
   ```bash
   mkfs.ext4 /dev/vg_lab/lv_data
   ```

3. **Create a mount point and mount the logical volume:**
   ```bash
   mkdir /mnt/diskl
   mount /dev/vg_lab/lv_data /mnt/diskl
   ```

4. **Create a logical volume named `lv_swap` with a size of 2GB and set it up as swap space:**
   ```bash
   lvcreate -L 2G -n lv_swap vg_lab
   mkswap /dev/vg_lab/lv_swap
   swapon /dev/vg_lab/lv_swap
   ```

5. **Create a logical volume named `lv_large` with a size of 5GB:**
   ```bash
   lvcreate -L 5G -n lv_large vg_lab
   ```
6. **Format the lv_large logical volume with the ext4 filesystem:**
   ```bash
   mkfs.ext4 /dev/vg_lab/lv_large
    ```
7. **Create a mount point and mount the logical volume:**
   ```bash
   mkdir /mnt/lv_large
   mount /dev/vg_lab/lv_large /mnt/lv_large


## Step 4: Extending a Logical Volume

1. **Extend the `lv_large` logical volume by 3GB:**
   ```bash
   lvextend -L +3G /dev/vg_lab/lv_large
   ```

2. **Resize the filesystem to use the new space:**
   ```bash
   resize2fs /dev/vg_lab/lv_large
   ```



## Step 5: Configuring Persistent Mount Points

1. **Add entries to `/etc/fstab` for persistent mounting:**
   ```bash
   echo "/dev/vg_lab/lv_data /mnt/diskl ext4 defaults 0 0" >> /etc/fstab
   echo "/dev/vg_lab/lv_swap swap swap defaults 0 0" >> /etc/fstab
   echo "/dev/vg_lab/lv_large /mnt/lv_large ext4 defaults 0 0" >> /etc/fstab
   ```


2. **Mount all filesystems listed in `/etc/fstab`:**
   ```bash
   mount -a
   ```

