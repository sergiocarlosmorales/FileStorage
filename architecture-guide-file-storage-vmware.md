---

copyright:
  years: 2014, 2017
lastupdated: "2017-08-24"

---
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}

# Architecture Guide for File Storage with VMware

Following are the steps to order and configure File Storage in a vSphere 5.5 and vSphere 6.0 environment at IBM Cloud Infrastructure.

## File Storage Overview

Our File Storage is designed to support high I/O applications requiring predictable levels of performance. The predictable performance is achieved through the allocation of protocol-level input/output operations per second (IOPS) to individual volumes. 

The file storage offering is accessed and mounted through an NFS connection. In a VMware deployment, a single volume can be mounted to up to 64 ESXi hosts as shared storage, or you can mount multiple volumes to create a storage cluster to utilize vSphere Storage DYNAMIC Resource Scheduling.

Pricing and configuration options for Endurance and Performance file storage are charged based on a combination of the reserved space and for the offered IOPS.

### 1. File Storage Considerations

When ordering file storage, you will want to keep in mind the following information and considerations:

- The storage size, IOPS, and operating system cannot be changed once file storage volumes are provisioned. Any changes you want to make for the amount of space, the number of IOPS, or the operating system requires a new volume to be provisioned. Any data stored in the previous volume will have to be migrated to the new volume(s) using VMware Storage vMotion.
- An administrator can choose the size of the instance needed for a workload. IOPS are measured based on a 16KB block size with a 50/50 read/write mix. This formula proves to be a very consistent calculation for block storage.
- NFS utilizes many additional file control operations such as lookup, getattr and readdir to name a few. These operations in addition to read/write operations can count as IOPS and vary by operation type and NFS version.
- Technically, multiple volumes can be striped together to achieve higher IOPS and more throughput, however, VMware recommends a single Virtual Machine File System (VMFS) data store per volume to avoid performance degradation.
- File Storage volumes are exposed to authorized devices, subnets, or IP addresses.
- Snapshot and Replication services are natively available on Endurance file storage volumes only. Performance file storage does not have these capabilities.

**Note**: Increasing block size will increase throughput but decrease IOPS. For example, doubling the block size to 32KB blocks will maintain the maximum throughput but halve the IOPS.

 

### 2. Endurance File Storage snapshots

Endurance file storage allows administrators to set snapshot schedules that create and delete snapshot copies automatically for each storage volume. They can also create additional snapshot schedules (hourly, daily, weekly) for automatic snapshots and manually create adhoc snapshots for business continuity and disaster recovery (BCDR) scenarios. Automatic alerts are delivered via the SoftLayer customer portal to the volume owner for the retained snapshots and space consumed.

Be aware that “snapshot space” is required to use snapshots. Space can be acquired on the initial volume ordering or after initial provisioning via the **Volume Details** page by clicking the Actions drop-down button and selecting **Add Snapshot Space**.

It is important to note that VMware environments are not aware of snapshots. The Endurance file storage snapshot capability must not be confused with VMware snapshots. Any recovery using the Endurance file storage snapshot feature must be handled from the SoftLayer customer portal. Restoring the Endurance file storage volume will require powering off all the VMs that reside on Endurance file storage, and temporarily unmounting the volume from the ESXi hosts to avoid any data corruption during the process.

Refer to our [snapshots](snapshots.html) article for more details about how to configure snapshots.
 

### 3. File Store Replication

Replication uses one of your snapshot schedules to automatically copy snapshots to a destination volume in a remote data center. The copies can be recovered in the remote site in the event of corrupted data or a catastrophic event.


Replicas let you

- Recover from site failures and other disasters quickly by failing over to the destination volume
- Failover to a specific point in time in the DR copy

Before you can replicate, you must create a snapshot schedule. When you failover, you are “flipping the switch” from your storage volume in your primary data center to the destination volume in your remote data center. For example, your primary data center is London and your secondary data center is Amsterdam. In the case of a failure event, you’d fail over to Amsterdam – connecting to the now-primary volume from a vSphere Cluster instance in Amsterdam.


After your volume in London has been repaired, a snapshot is taken of the Amsterdam volume in order to fall back to London and the once-again primary volume from a compute instance in London. Before the volume fails back to the primary data center, it needs to stop being used at the remote site. A snapshot of any new or changed information is taken and replicated to the primary data center before it can be mounted again on the production site ESXi hosts.


Refer to the [Replication](replication.html) information page for more details about how to configure replication.

**Note**: Invalid data, whether corrupted, hacked or infected will replicate according to the snapshot schedule and snapshot retention. Using the smallest replication windows can provide for a better recovery point objective. It also may provide less time to react to the replication of invalid data.

 

 

## Order File Storage

You can order and configure File storage for a VMware ESXi 5 environment. Use the following information in conjunction with the Advanced Single-Site VMware Reference Architecture to set up one of these storage options in your VMware environment.


File storage can be ordered through the SoftLayer customer portal (control.SoftLayer.com) by accessing the File Storage page (Storage, File Storage). The process for ordering Endurance and Performance file storage is slightly different.

 

### 1. Ordering File Storage

Use the following steps to order File storage:
1. Click **Storage** > **File Storage** from the [Customer portal](https://control.softlayer.com/) home page.
2. Click on the **Order File Storage** link on the **File Storage** page.
3. Select the desired amount of storage space in GBs. For TB, 1TB equals 1,000GB, and 12TB equals 12,000GB.
4. Enter the desired amount of IOPS in intervals of 100 or select an IOPS Tier.
5. Submit the order.

Storage will be provisioned in less than a minute and will be visible on the **File Storage** page of the SoftLayer customer portal.

 

### 2. Authorize Hosts to use File Storage

Once a volume is provisioned, the bare metal servers or virtual machines that will use the volume must be authorized to access the storage. Use the following steps to authorize the volume:

1. Click on **Storage** > **File Storage**.
2. Select **Access Host** on the **Endurance** or **Performance Volume Actions** menu.
3. Select the **Subnets** radio button
4. Choose from the list of available subnets that are assigned to the vmkernel ports on the ESXi hosts, and click on **Submit**.

After the subnets are authorized, make note of the hostname of the Endurance or Performance storage server you wish to utilize when mounting the volume. This information can be found on the file storage detail page by clicking on a specific volume.

 
##  Configure the VMware Virtual Machine Host

### 1. Prerequisites

Before beginning the VMware configuration process, make sure that the following prerequisites are met:

- IBM Cloud (SoftLayer) bare metal server with VMware ESXi are provisioned with proper storage configuration and ESXi login credentials.
- IBM Cloud (SoftLayer) Windows physical or virtual machine in the same data center as the bare metal server. Including Public IP address of the IBM Cloud (SoftLayer) Windows VM and login credentials.
- A computer with Internet access, and with the web browser software and a Remote Desktop Protocol (RDP) client installed.
 

### 2. VMware Host Configuration Steps.

To configure the virtual host, complete the following steps:

1. From an Internet connected computer, launch an RDP client and establish an RDP session to the SoftLayer Windows VM provisioned in the same data center where vSphere vCenter is installed.
2. From the SoftLayer Windows virtual machine, launch a web browser and connect to VMware vCenter via the vSphere Web Client.
3. From the **HOME** screen select **Hosts and Clusters**. Expand the panel on the left and select the **VMware ESXi server** that is to be used for this deployment.
4. Make sure the firewall port for the NFS client is open on all hosts in order to configure the NFS client on the vSphere host. This is automatically opened in the more recent releases of vSphere. To check if the port is open, go to the **ESXi host Manage** tab in VMware® vCenter™, select **Settings** and then select **Security Profile**. In the **Firewall** section click **Edit** and scroll down to **NFS Client**.
5. Make sure **Allow connection from any IP address or a list of IP addresses** is provided. <br/>
   ![Allow Connection](/images/1_4.png)
6. Configure Jumbo Frames by going to the **ESXi host Manage** tab, select **Manage** and then **Networking**.
7. Select **VMkernel adapters**, highlight the **vSwitch** and the click **Edit**. (Pencil icon)
8. Select **NIC setting** and ensure the NIC MTU is set to 9000.
   - If required, set the MTU to 9000 and Click **OK**.
9. If desired, the jumbo frame settings can be validated as follows:
   - From Windows: ping -f -l 8972 a.b.c.d
   - From Unix: ping -s 8972 a.b.c.d
   Where a.b.c.d is neighboring virtual machine interface with the command:
   The output appears similar to:
   ```ping a.b.c.d (a.b.c.d) 8972(9000) bytes of data.
   8980 bytes from a.b.c.d: icmp_seq=1 ttl=128 time=3.36 ms
   ```

More information on VMware and Jumbo Frames can be found here: https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1003712
 

### 3. Adding An Uplink Adapter To A Virtual Switch

1. Configure a new uplink adapter by going to the **ESXi host Manage** tab, select **Manage** and then **Networking**.
2. Select the **Physical adapters** tab 
3. Click **Add host networking**. (Globe icon with a plus sign)
4. Select connection type as **Physical Network Adapter** and click **Next**.
5. Select the existing **vSwitch** and click **Next**.
6. Select **Unused adapters** and click **Add adapters**. (Plus sign)
7. Click the other "Connected" adapter and click **OK**. <br/>
   ![Add Physical Adapters to Switch](/images/2_3.png)
8. Click **Next** and the **Finish**.
9. Navigate back to the **Virtual switches** tab and select the upper **Edit setting** icon under the **Virtual Switches** heading. (Pencil icon)
10. Select the **vSwitch Teaming** and failover entry on the left.
Verify that the **Load balancing** option is set to **Route based on the originating virtual port** and click **OK**.
 

### 4. Configure ESXi static routing (Optional)

The network configuration for this architecture guide uses a minimal number of port groups. If you have set up a VMkernal port group for NFS storage additional steps must be performed. By default, ESXi will use the vmkernel port that is on the same subnet as an NFS volume to mount the NFS volume on Endurance/Performance storage. Since we’re using layer 3 routing to mount the NFS volume, we must force ESXi to use the vmkernel port we configured to mount the NFS volume. To do this, we must create a static route to Endurance/Performance storage array.

1. To configure a static route, SSH to each ESXi host utilizing Performance or Endurance and run the following commands. Note that you must take the resulting IP address  ping command (first command) and use it with the esxcli network command shown below:
   - `ping <hostname of the endurance/performance array>`
   
      **Note** that the NFS storage DNS host name is a Forwarding Zone (FZ) that is assigned multiple IP addresses. These IP addresses are static and belong to that specific DNS host name.  Any of those IP addresses can be used to access a specific volume.
   - `esxcli network ip route ipv4 add –gateway GATEWAYIP –network <result of ping command>/32`

2. Note that static routes are not persistent across reboots on ESXi 5.0 and earlier. To ensure that any added static routes are persistent, these commands need to be added to the local.sh file on each host, located in the /etc/rc.local.d/ directory. To do this, open the local.sh file using the visual editor and add the command above to be executed above the line exit 0.
   - Make note of the IP address as it can be used for mounting the volume in the next step.
   - This process needs to be done for each NFS volume you plan to mount to your ESXi host.
   - Here is the link to a VMware KB article [Configuring static routes for VMkernel ports on an ESXi host](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2001426).
 

##  Mount File Storage Volume(s) on the ESXi hosts

1. Click **Go to vCenter** icon at the top of the web page and then **Hosts and Clusters**.
2. Click on **Datastores** under the **Related Object** tab. Click on the **Create a new datastore** icon. 
3. On the **New Datastore** screen, select the location of the datastore (your ESXi host) and click **Next**.
4. On the **Type** screen, select **NFS** radio button, and click **next**.
5. On the **Name and configuration** screen, enter the name you wish to call the datastore. Additionally, enter the hostname of the NFS server. Using the FQDN for the NFS server produces the best traffic distribution to the underlying server. IP address is also valid but used less frequently and only in specific instances. Enter the folder name in the form of /foldername.
6. On the **Host accessibility** screen, select all the host(s) that you wish to mount the NFS datastore and click **next**.
7. Review the inputs on the next screen and click **Finish**.
8. Repeat for any additional File storage volumes.

**Note**: It is IBM Cloud’s recommendation that FQDN names be used to connect to the datastore. Using direct IP addressing may be bypassing the load balancing mechanism provided by using FQDN. To use the IP address instead of the FQDN execute the following command to obtain the IP address.

  - `ping <hostname of the endurance/performance array>`
  - From an ESXi host:
    ```
    ~ # vmkping nfsdal0902a-fz.service.softlayer.com
    PING nfsdal0902a-fz.service.softlayer.com (10.2.125.80): 56 data bytes
    64 bytes from 10.2.125.80: icmp_seq=0 ttl=253 time=0.187 ms
    10.2.125.80 is the IP address associated with the FQDN
    ```

## Enable ESXi Storage I/O Control settings (Optional)

Storage I/O Control (SIOC) is a feature available for customers utilizing an Enterprise Plus license. When SIOC is enabled in the environment, it will change the device queue length for the single VM. The change to the device queue length reduces the storage array queue for all VMs to an equal share and throttle of the storage queue. SIOC engages only if resources are constrained and the storage I/O latency is above a defined threshold.


In order for SIOC to determine when a storage device is congested or constrained, it requires a defined threshold. The congestion threshold latency is different for different storage types; the default selections is to 90% of peak throughput. The percentage of peak throughput value indicates the estimated latency threshold when the datastore is using that percentage of its estimated peak throughput.


It should be noted that incorrectly configuring SIOC for a datastore or for a VMDK can significantly impact performance.

 

### 1. Storage I/O Control For A Datastore

Use the following steps to enable SIOC with recommended values for Endurance and Performance Storage:

1. Browse to the datastore in the vSphere Web Client navigator.
2. Click the **Manage** tab.
3. Click **Settings** and click **General**.
4. Click **Edit** for **Datastore Capabilities**.
5. Select the E**nable Storage I/O Control** check box.<br/>
   ![NSFDataStore](/images/3_0.png)
6. Click **OK**.

**Note**: This setting is specific to the datastore and not to the host.
 

### 2. Storage I/O Control For A Virtual Machine

You can also limit individual virtual disks for individual VMs or grant them different shares with SIOC. The limiting of disks and granting different shares allows you to match and align the environment to the workload with the acquired file storage volume IOPS number. The limit is set by IOPS and it is possible to set a different weight or "Shares." Virtual disks shares set to High (2,000 shares) receive twice as much as I/O as a disk set to Normal (1,000 shares) and four times as much as one set to Low (500 shares). Normal is the default value for all the VMs, so you just need to adjust the values above or below Normal for the VMs that actually require it.


Use the following steps to change the vDisk shares and limit:

1. Choose a virtual machine in the **VMs and Templates** inventory. Icon is on the top left of the web page. 
2. Select the virtual machine for I/O Control.
3. Click the **Manage** tab and click **Settings**. Click **Edit**.
4. Expand the Hard disk pull down arrow. Modify the Shares or Limit-IOPs as is appropriate for your environment. Choose a virtual hard disk from the list and modify the Shares selection to choose the relative amount of shares to allocate to the virtual machine (Low, Normal, or High). You can also click **Custom** and enter a user-defined share value.
5. Click the Limit - IOPS column and enter the upper limit of storage resources to allocate to the virtual machine.
6. Click **OK**
 

   **Note**: The above process is used to set the resource consumption limits of individual vDisks in a virtual machine even when SIOC is not enabled. These settings are specific to the individual guest, and not the host, although they are used by SIOC.

 

 

## ESXi  Host Side Settings

There are some additional settings required for setting up ESXi 5.x hosts for NFS storage.

|Parameter | Set to ... |
|----------|------------|
|Net.TcpipHeapSize |	32 |
|Net.TcpipHeapMax |	For vSphere 5.0/5.1 set 128 <br/> For vSphere 5.5 or higher set 512 |
|NFS.MaxVolumes |	256 |
|NFS41.MaxVolumes |	256 (vSphere 6.0 or later only) |
|NFS.HeartbeatMaxFailures |	10 |
|NFS.HeartbeatFrequency |	12 |
|NFS.HeartbeatTimeout |	5 |
|NFS.MaxQueueDepth|	64 |
 

- Using CLI on ESXi 5.x host for advanced configuration parameters:
  The following examples use the CLI to set these advanced configuration parameters and then check them. The esxcfg-advcfg tool used in the examples can be found in the /usr/sbindirectory on the ESXi 5.x hosts.

   - Setting the advanced configuration parameters from the ESXi 5.x CLI:
   ```
   #esxcfg-advcfg -s 32 /Net/TcpipHeapSize
   #esxcfg-advcfg -s 128 /Net/TcpipHeapMax(For vSphere 5.0/5.1)
   #esxcfg-advcfg -s 512 /Net/TcpipHeapMax(For vSphere 5.5 and above)
   #esxcfg-advcfg -s 256 /NFS/MaxVolumes
   #esxcfg-advcfg -s 256 /NFS41/MaxVolumes (ESXi 6.0 and above)
   #esxcfg-advcfg -s 10 /NFS/HeartbeatMaxFailures
   #esxcfg-advcfg -s 12 /NFS/HeartbeatFrequency
   #esxcfg-advcfg -s 5 /NFS/HeartbeatTimeout   
   #esxcfg-advcfg -s 64 /NFS/MaxQueueDepth
   #esxcfg-advcfg -s 32 /Disk/QFullSampleSize
   #esxcfg-advcfg -s 8 /Disk/QFullThreshold
   ```

- Checking the advanced configuration parameters from the ESXi 5.x CLI:
   ```
   #esxcfg-advcfg -g /Net/TcpipHeapSize
   #esxcfg-advcfg -g /Net/TcpipHeapMax
   #esxcfg-advcfg -g /NFS/MaxVolumes
   #esxcfg-advcfg -g /NFS41/MaxVolumes (ESXi 6.0 and above)
   #esxcfg-advcfg -g /NFS/HeartbeatMaxFailures
   #esxcfg-advcfg -g /NFS/HeartbeatFrequency
   #esxcfg-advcfg -g /NFS/HeartbeatTimeout
   #esxcfg-advcfg -g /NFS/MaxQueueDepth
   #esxcfg-advcfg -g /Disk/QFullSampleSize
   #esxcfg-advcfg -g /Disk/QFullThreshold
   ```


## Enabling Jumbo Frames in SoftLayer – Windows and Linux

Softlayer has stated that in order to fully realize the speeds on the storage Jumbo Frames needs to be enabled at 9,000 MTU.


A jumbo frame is an Ethernet frame with a payload greater than the standard maximum transmission unit (MTU) of 1,500 bytes. Jumbo frames are used on local area networks that support at least 1 Gbps and can be as large as 9,000 bytes.


Jumbo Frames needs to be configured the same on the entire network path from source device <-> switch <-> router <-> switch <-> destination device. If the entire chain is not set the same it will default to the lowest setting along the chain. SoftLayer has their network devices set to 9,000 currently. All customer devices will need to be set to the same.

### Windows

1. Open the **Network and Sharing Center**.
2. Click **Change adapter settings**.
3. Right-click the NIC for which you want to enable jumbo frames and select **Properties**.
4. Under the **Networking** tab, click **Configure** for the network adapter.
5. Select the **Advanced** tab.
6. Select **Jumbo Frame** and change the value from disabled to the desired value, such as 9kB MTU or 9,014 Bytes, depending on the NIC.
7. Click **OK** to all dialogs.

Note that when you make the change, the NIC will lose network connectivity for a few seconds. You should also reboot to ensure the change has taken effect.

 

### LINUX

1. Edit the network configuration file for eth0 interface – for example, /etc/sysconfig/network-script/ifcfg-eth0 (CentOS / RHEL / Fedora Linux):
`# vi /etc/sysconfig/network-script/ifcfg-eth0`

2. Append the following configuration directive, which specifies the size of the frame in bytes:
MTU 9000

3. Close and save the file. Restart the Interface eth0:
`# /etc/init.d/networking restart (This will cause a brief loss of network connectivity)`


A note about Debian / Ubuntu Linux user:
Debian / Ubuntu Linux user should add MTU=9000 to /etc/network/interfaces configuration file.

Learn more about Advanced Single-Site VMware Reference Architecture [here](http://knowledgelayer.softlayer.com/articles/advanced-single-site-vmware-reference-architecturesoftlayer).
