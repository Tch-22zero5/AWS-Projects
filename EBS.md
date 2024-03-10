# Elastic Block Store 
how to create an Amazon EBS volume, attach it to an instance, apply a file system to the volume, and then take a snapshot backup.
## What is Amazon Elastic Block Store?
Amazon Elastic Block Store (Amazon EBS) offers persistent storage for Amazon EC2 instances. Amazon EBS volumes are network-attached and persist independently from the life of an instance. Amazon EBS volumes are highly available, highly reliable volumes that can be leveraged as an Amazon EC2 instances boot partition or attached to a running Amazon EC2 instance as a standard block device.

When used as a boot partition, Amazon EC2 instances can be stopped and subsequently restarted, enabling you to pay only for the storage resources used while maintaining your instance's state. Amazon EBS volumes offer greatly improved durability over local Amazon EC2 instance stores because Amazon EBS volumes are automatically replicated on the backend (in a single Availability Zone).

For those wanting even more durability, Amazon EBS provides the ability to create point-in-time consistent snapshots of your volumes that are then stored in Amazon Simple Storage Service (Amazon S3) and automatically replicated across multiple Availability Zones. These snapshots can be used as the starting point for new Amazon EBS volumes and can protect your data for long-term durability. You can also easily share these snapshots with co-workers and other AWS developers.

This lab guide explains basic concepts of Amazon EBS in a step-by-step fashion. However, it can only give a brief overview of Amazon EBS concepts. For further information, see the Amazon EBS documentation.

Amazon EBS Volume Features
Amazon EBS volumes deliver the following features:

Persistent storage: Volume lifetime is independent of any particular Amazon EC2 instance.

General purpose: Amazon EBS volumes are raw, unformatted block devices that can be used from any operating system.

High performance: Amazon EBS volumes are equal to or better than local Amazon EC2 drives.

High reliability: Amazon EBS volumes have built-in redundancy within an Availability Zone.

Designed for resiliency: The AFR (Annual Failure Rate) of Amazon EBS is between 0.1% and 1%.

Variable size: Volume sizes range from 1 GB to 16 TB.

Easy to use: Amazon EBS volumes can be easily created, attached, backed up, restored, and deleted.
# Task 1: Create a New EBS Volume
In the AWS Management Console, on the Services menu, click EC2.

In the left navigation pane, choose Instances. ( Select the Ec2 instance you wish to attach the EBS)
Note the Availability Zone of the instance. 
In the left navigation pane, choose Volumes.

You will see an existing volume that is being used by the Amazon EC2 instance. This volume has a size of 8 GiB, which makes it easy to distinguish from the volume you will create next, which will be 1 GiB in size.

Choose Create volume then configure:

Volume Type: General Purpose SSD (gp2)

Size (GiB): 1. NOTE: You may be restricted from creating large volumes.

Availability Zone: Select the same availability zone as your EC2 instance.

Choose Add Tag

In the Tag Editor, enter:

Key: Name

Value: My Volume

Choose Create Volume

Your new volume will appear in the list, and will move from the Creating state to the Available state. You may need to choose refresh  to see your new volume.
# Task 2: Attach the Volume to an Instance
You can now attach your new volume to the Amazon EC2 instance.

Select  My Volume. 

In the Actions menu, choose Attach volume.

Choose the Instance field, then select the instance that appears 
Note that the Device field is set to /dev/sdf. You will use this device identifier in a later task.

Choose Attach volume
The volume state is now In-use.
# Task 3: Connect to Your Amazon EC2 Instance
# Windows Users: Using SSH to Connect
These instructions are for Windows users only.

If you are using macOS or Linux, skip to the next section.

Read through the three bullet points in this step before you start to complete the actions, because you will not be able see these instructions when the Details panel is open.

Choose the Details drop down menu above these instructions you are currently reading, and then choose Show. A Credentials window will open.

Choose the Download PPK button and save the labsuser.ppk file. Typically your browser will save it to the Downloads directory.

Then exit the Details panel by choosing the X. 

Download needed software.

You will use PuTTY to SSH to Amazon EC2 instances. If you do not have PuTTY installed on your computer, download it here.

Open putty.exe

Configure PuTTY to not timeout:

Choose Connection
Set Seconds between keepalives to 30
This allows you to keep the PuTTY session open for a longer period of time.

Configure your PuTTY session:

Choose Session
Host Name (or IP address): Paste the Public DNS or IPv4 address of the Lab instance that you noted earlier. 
Back in PuTTY, in the Connection list, expand  SSH
Choose Auth and expand  Credentials
Under Private key file for authentication: Choose Browse
Browse to the labsuser.ppk file that you downloaded, select it, and choose Open
Choose Open again
To trust and connect to the host, choose Accept.
When prompted login as, enter: ec2-user

This will connect you to the EC2 instance.

# Windows Users: Choose here to skip ahead to the next task.
# macOS  and Linux  Users
These instructions are for Mac/Linux users only. If you are a Windows user, skip ahead to the next task. 

Read through all the instructions in this one step before you start to complete the actions, because you will not be able see these instructions when the Details panel is open.

Choose the Details drop down menu above these instructions you are currently reading, and then choose Show. A Credentials window will open.

Choose the Download button and save the labsuser.pem file.

Then exit the Details panel by choosing the X.

Open a terminal window, and change directory cd to the directory where the labsuser.pem file was downloaded.

For example, run this command, if it was saved to your Downloads directory:

cd ~/Downloads
 
Change the permissions on the key to be read only, by running this command:

chmod 400 labsuser.pem
 
Return to the AWS Management Console, and in the EC2 service, choose Instances.

The Lab instance should be selected. 

In the Details tab, copy the Public IPv4 address value.

Return to the terminal window and run this command (replace <public-ip> with the actual public IP address you copied):

ssh -i labsuser.pem ec2-user@<public-ip>
 
Type yes when prompted to allow a first connection to this remote SSH server.

Because you are using a key pair for authentication, you will not be prompted for a password.
# Task 4: Create and Configure Your File System
In this task, you will add the new volume to a Linux instance as an ext3 file system under the /mnt/data-store mount point.

If you are using PuTTY, you can paste text by right-clicking in the PuTTY window.

 

View the storage available on your instance:

df -h
You should see output similar to:

Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        484M     0  484M   0% /dev
tmpfs           492M     0  492M   0% /dev/shm
tmpfs           492M  460K  491M   1% /run
tmpfs           492M     0  492M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.5G  6.6G  19% /
tmpfs            99M     0   99M   0% /run/user/0
tmpfs            99M     0   99M   0% /run/user/1000
This is showing the original 8GB disk volume. Your new volume is not yet shown.

 

Create an ext3 file system on the new volume:

sudo mkfs -t ext3 /dev/sdf
 

Create a directory for mounting the new storage volume:

sudo mkdir /mnt/data-store
 

Mount the new volume:

sudo mount /dev/sdf /mnt/data-store
To configure the Linux instance to mount this volume whenever the instance is started, you will need to add a line to /etc/fstab.

echo "/dev/sdf   /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab
 

View the configuration file to see the setting on the last line:

cat /etc/fstab
 

View the available storage again:

df -h
The output will now contain an additional line - /dev/xvdf:

Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        484M     0  484M   0% /dev
tmpfs           492M     0  492M   0% /dev/shm
tmpfs           492M  460K  491M   1% /run
tmpfs           492M     0  492M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.5G  6.6G  19% /
tmpfs            99M     0   99M   0% /run/user/0
tmpfs            99M     0   99M   0% /run/user/1000
/dev/xvdf       976M  1.3M  924M   1% /mnt/data-store
 

On your mounted volume, create a file and add some text to it.

sudo sh -c "echo some text has been written > /mnt/data-store/file.txt"
 

Verify that the text has been written to your volume.

cat /mnt/data-store/file.txt
# Task 5: Create an Amazon EBS Snapshot
You can create any number of point-in-time, consistent snapshots from Amazon EBS volumes at any time. Amazon EBS snapshots are stored in Amazon S3 with high durability. New Amazon EBS volumes can be created out of snapshots for cloning or restoring backups. Amazon EBS snapshots can also be easily shared among AWS users or copied over AWS regions. 

In the AWS Management Console, choose Volumes and select  My Volume.

In the Actions menu, select  Create snapshot. 

Choose Add tag then configure:

Key: Name
Value: My Snapshot
Choose Create snapshot
    
In the left navigation pane, choose Snapshots.

Your snapshot is displayed. The status will first have a state of Pending, which means that the snapshot is being created. It will then change to a state of Completed. 

Note: Only used storage blocks are copied to snapshots, so empty blocks do not occupy any snapshot storage space.

In your remote SSH session, delete the file that you created on your volume.

sudo rm /mnt/data-store/file.txt
 
Verify that the file has been deleted.

ls /mnt/data-store/
Your file has been deleted.
# Task 6: Restore the Amazon EBS Snapshot
If you ever wish to retrieve data stored in a snapshot, you can Restore the snapshot to a new EBS volume.

# Create a Volume Using Your Snapshot
In the AWS Management Console, select  My Snapshot. 

In the Actions menu, select Create volume from snapshot.

For Availability Zone Select the same availability zone that you used earlier. 

Choose Add tag then configure:

Key: Name
Value: Restored Volume
Choose Create volume
Note: When restoring a snapshot to a new volume, you can also modify the configuration, such as changing the volume type, size or Availability Zone.
# Attach the Restored Volume to Your EC2 Instance
In the left navigation pane, choose Volumes.

Select  Restored Volume.

In the Actions menu, select Attach volume. 

Choose the Instance field, then select the (Lab) instance that appears.

Note that the Device field is set to /dev/sdg. You will use this device identifier in a later task. 

Choose Attach volume

The volume state is now in-use.

# Mount the Restored Volume
Create a directory for mounting the new storage volume:

sudo mkdir /mnt/data-store2
 
Mount the new volume:

sudo mount /dev/sdg /mnt/data-store2
 
Verify that volume you mounted has the file that you created earlier.

ls /mnt/data-store2/
You should see file.txt.
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/95f19cc4-3ddc-4b38-9239-fca2721ae0ba)
Congratulations! You now have successfully:

Created an Amazon EBS volume

Attached the volume to an EC2 instance

Created a file system on the volume

Added a file to volume

Created a snapshot of your volume

Created a new volume from the snapshot

Attached and mounted the new volume to your EC2 instance

Verified that the file you created earlier was on the newly created volume.
