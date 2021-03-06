---

copyright:
  years: 2014, 2017
lastupdated: "2017-06-24"

---
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}

# Migrate File Storage to Encrypted File Storage


## Overview

Encrypted file storage for Endurance or Performance has been launched in select data centers. Below you will find information on how to migrate your file storage from unencrypted to encrypted. For more information on provider managed encrypted storage, read Block and File Storage Encryption-At-Rest article. To see a list of upgraded data centers and available features click here.

The preferred migration path is to connect to both volumes simultaneously and transfer data directly from one file volume to another. The specifics will depend on your operating system  and whether the data is expected to change during the copy operation.

The more common scenarios have been outlined for your convenience. There is an assumption that you already have your non-encrypted file volume attached to your host. If not, follow the directions below that best fits the operating system you're running to accomplish this task. 

**NOTE:**  All encrypted File storage volumes have a different mount point than non-encrypted volumes.  To ensure you are using the correct mount point for both your encrypted and non-encrypted file storage volumes you can view the mount point information in the Volume Detials page in the UI as well as access the correct mountpoint via an API call:  SoftLayer_Network_Storage::getNetworkMountAddress().

[Accessing File Storage on Linux](accessing-file-storage-linux.html)

## Create an Encrypted File Volume

Use the following steps to create a volume the same size or larger that is encrypted to facilitate the migration process.

### Order an Encrypted Endurance Storage Volume

1. Click **Storage** > **File Storage** from the Bluemix customer portal home page OR Click **Infrastructure** > **Storage** > **File Storage** in the Bluemix catalog.

2. Click **Order File Storage** link on the File Storage page.

3. Select **Endurance**.

4. Select the data center where your original volume is located. Note that encryption is only available in data centers with an asterisk.

5. Enter the desired **IOPS tier**.

6. Select the desired amount of storage space in GBs. For TB, 1TB equals 1,000GB, and 12TB equals 12,000GB.

7. Enter the desired amount of storage space in GBs for snapshots.

8. Select the **VMware OS** from the drop-down list.

9. Submit the order.
 
### Order an Encrypted Performance Storage Volume

1. Click **Storage** > **File Storage** from the Customer portal home page OR Click **Infrastructure** > **Storage** > **File Storage** in the Bluemix catalog.

2. Click on the **Order File Storage**.

3. Select **Performance**.

4. Select the data center where your original volume is located. Note that encryption is only available in data centers with an asterisk.

5. Select the desired amount of storage space in GBs of the same size of the original volume or larger.

6. Enter the desired amount of IOPS in intervals of 100.

7. Select the VMware OS from the drop-down list.

8. Submit the order.

Storage will be provisioned in less than a minute and will be visible on the File Storage page of the customer portal.

 
## Connect New Volume to Host

“Authorized” hosts are hosts that have been given access rights to a volume. Without host authorization, you won’t be able to access or use the storage from your system.

1. Click on your encrypted **Volume Name**.

2. Scroll to the **Authorized Hosts** section of the page.

3. Click the **Authorize Host** link on the right side of the page. Select the hosts that can access the volume.

Once Authorized, connect the volume to your host.

 
## Snapshots and Replication

Do you have snapshots and replication established for your original volume? If yes, you will need to set up replication, snapshot space and create snapshot schedules for the new encrypted volume with the same settings as the original volume. 

Note that if your target data center has not been upgraded for encryption, you will not be able to establish replication for the new volume until that data center is upgraded.

 
## Migrate your data

Your host should be connected to both your original and encrypted file storage volumes. If not:

• Make sure that you have followed the steps above and referenced documents correctly.

• Open a support ticket for further assistance in connecting the two volumes to your host.

### Data considerations

At this point, consider what type of data you have on your original file storage volume and how best to copy it to your encrypted volume. If you have backups, static content, and things that aren't expected to change during the copy, there aren't any major considerations.

If you're running a database or a virtual machine on your file storage, make sure that the data on the original volume is not altered during copy so that no corruption occurs. If you have any bandwidth concerns, you should perform the migration during off peak times. If you need assistance with these considerations, please do not hesitate to open a support ticket.

### Microsoft Windows

To copy data from your original file storage volume to your encrypted volume, format the new storage and copy the files over using Windows Explorer.

### Linux

You may consider using rsync to copy over the data. Below is an example command

`[root@server ~]# rsync -Pavzu /path/to/original/file/storage/* /path/to/encrypted/file/storage` 

It’s recommended that you use the above command with the `--dry-run` flag once to make sure the paths line up correctly. If this process is interrupted, you may want to delete the last destination file that was being copied to make sure that it is copied from the beginning to the new location.

Once this command completes without the `--dry-run` flag, your data should be copied to the encrypted file storage volume. You should scroll up and run the command again to make sure nothing was missed. You may also want to manually review both locations to look for anything that might be missing.

When your migration is complete, you will be able to move production to the encrypted volume and detach and delete your original volume from your configuration. Note that the deletion will also remove any snapshot or replica on the target site that was associated with the original volume.
