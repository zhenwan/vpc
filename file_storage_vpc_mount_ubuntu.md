---

copyright:
  years: 2021, 2023
lastupdated: "2023-01-27"

keywords: mounting VPC file storage, mount file storage,

subcollection: vpc

---

{{site.data.keyword.attribute-definition-list}}

# Mounting file shares on Ubuntu
{: #file-storage-vpc-mount-ubuntu}

Use these instructions to connect a Network File System (NFS) file share to an Ubuntu Linux&reg;-based {{site.data.keyword.cloud}} Compute instance.
{: shortdesc}

{{site.data.keyword.filestorage_vpc_full}} is available for customers with special approval to preview this service in the Frankfurt, London, Dallas, Toronto, Washington, Sao Paulo, Sydney, Osaka, and Tokyo regions. Contact your IBM Sales representative if you are interested in getting access.
{: preview}

## Before you begin - Create a VSI
{: #fs-ubuntu-create-vsi}

Before you mount {{site.data.keyword.filestorage_vpc_short}} file shares, you must create a [virtual server instance](/docs/vpc?topic=vpc-about-advanced-virtual-servers) in the same zone as the file share. After you created the instance, get the mount path of the file share from the mount target that was created.

Mount path information can be obtained from the File share details page in the UI, or through an API or CLI call.
{: tip}

## Mount the file share
{: #fs-Ubuntu-mount}

SSH into the virtual server instance where you want to mount the file share, then continue with the following steps to mount a file share. This example procedure is based on Ubuntu 20.04.

VPC File Storage service requires NFS versions v4.1 or higher.
{: note}

1. Update and upgrade the distribution:

    ```zsh
    apt update && apt upgrade
    ```
    {: pre}

2. Create a `/mnt/nfs` directory.

    ```zsh
    mkdir -p /mnt/nfs
    ```
    {: pre}

3. Install `nfs-common`:

    ```zsh
    apt install nfs-common
    ```
    {: pre}

4. Restart your instance:

    ```zsh
    reboot
    ```
    {: pre}

5. Mount the remote file share:

   ```zsh
   mount -t nfs4 -o <options> <host:/mount_target> /mnt/nfs
   ```
   {: pre}

   See following example.

   ```zsh
   mount -t nfs4 -o sec=sys,nfsvers=4.1 fsf-dal2433a-dz.adn.networklayer.com:/nxg_s_voll_mz0726_c391f0ba-50ed-4460-8704-a36032c96a4c /mnt/nfs
   ```
   {: pre}

6. Verify that the mount was successful by using the disk file system command `df -h`:

    ```zsh
    $ df -h
    Filesystem                                                                                    Size  Used Avail Use% Mounted on
    /dev/root                                                                                      97G  1.6G   96G   2% /
    devtmpfs                                                                                      3.9G     0  3.9G   0% /dev
    tmpfs                                                                                         3.9G     0  3.9G   0% /dev/shm
    tmpfs                                                                                         798M  508K  797M   1% /run
    tmpfs                                                                                         5.0M     0  5.0M   0% /run/lock
    tmpfs                                                                                         3.9G     0  3.9G   0% /sys/fs/cgroup
    /dev/vda15                                                                                    105M  9.2M   96M   9% /boot/efi
    /dev/loop0                                                                                     56M   56M     0 100% /snap/core18/1885
    /dev/loop1                                                                                     71M    71M     0 100% /snap/lxd/16922
    /dev/loop2                                                                                     31M   31M     0 100% /snap/snapd/9279
    tmpfs                                                                                         798M     0  798M   0% /run/user/0
    fsf-dal1099a-fz.adn.networklayer.com:/voll_58fd55a_685c_4ccd_b42e_25d5b61129e2   95G  256K   95G   1% /mnt/nfs
    ```

7. Go to the mount point to create a test file and list all files to verify that the share is mounted as read/write.

   ```zsh
   touch /mnt/nfs/test.txt
   ```
   {: pre}

   ```zsh
   ls -al /mnt/nfs
   ```
   {: pre}

   ```zsh
   touch /mnt/nfs/test.txt
   ls -al /mnt/nfs
   total 12
   drwxr-xr-x   2 nobody nobody 4096 Apr 28 15:52 .
   dr-xr-xr-x. 22 root   root   4096 Apr 28 14:30 ..
   -rw-r--r--   1 nobody nobody    0 Apr 28 15:52 test.txt
   ```

8. Make the configuration persistent by editing the file systems table (`/etc/fstab`). Add the remote share to the list of entries that are automatically mounted on startup:

   ```zsh
   sudo nano /etc/fstab
   ```
   {: pre}

   Add a line with the following syntax to the end of file.

   ```zsh
   (hostname):/(mount_point) /mnt nfs_version defaults 0 0
   ```

   Example

   ```zsh
   fsf-dal2433a-dz.adn.networklayer.com:/nxg_s_voll_mz0726_c391f0ba-50ed-4460-8704-a36032c96a4c /mnt nfsvers=4.1 defaults 0 0
   ```

9. Verify that the configuration file has no errors.

   ```zsh
   mount -fav
   ```
   {: pre}

   If the command completes with no errors, your setup is complete.

   For NFS 4.1, add `sec=sys` to the mount command to prevent file ownership issues. Use `_netdev` to wait for the storage to be mounted until after all network components are started.
   {: tip}

## Unmounting the file system
{: #fs-Ubuntu-umount}

To unmount any currently mounted file system on your host, run the `umount` command with disk name or mount point name.

```zsh
umount /dev/sdb
```
{: pre}

```zsh
umount /mnt/nfs
```
{: pre}
