# Ec2
## A basic overview of launching, resizing, managing, and monitoring an Amazon EC2 instance.
### Task 1: Launch Your Amazon EC2 Instance: will launch an Amazon EC2 instance with termination protection. Termination protection prevents you from accidentally terminating an EC2 instance. You will deploy your instance with a User Data script that will allow you to deploy a simple web server.
In the AWS Management Console choose  Services, choose Compute, and then choose EC2.

Note: Verify that your EC2 console is currently managing resources in the N. Virginia (us-east-1) region.  You can verify this by looking at the drop down menu at the top of the screen, to the left of your username. If it does not already indicate N. Virginia, choose the N. Virginia region from the region menu before proceeding to the next step.

Choose the Launch instance  menu and select Launch instance.

Step 1: Name and tags
Give the instance the name Web Server.

The Name you give this instance will be stored as a tag. 
# Step 2: Application and OS Images (Amazon Machine Image)
In the list of available Quick Start AMIs, keep the default Amazon Linux AMI selected. 

Also keep the default Amazon Linux 2023 AMI selected.

An Amazon Machine Image (AMI) provides the information required to launch an instance, which is a virtual server in the cloud. An AMI includes:

A template for the root volume for the instance (for example, an operating system or an application server with applications)

Launch permissions that control which AWS accounts can use the AMI to launch instances

A block device mapping that specifies the volumes to attach to the instance when it is launched

The Quick Start list contains the most commonly-used AMIs. You can also create your own AMI or select an AMI from the AWS Marketplace, an online store where you can sell or buy software that runs on AWS.

# Step 3: Instance type
In the Instance type panel, keep the default t2.micro selected.

Amazon EC2 provides a wide selection of instance types optimized to fit different use cases. Instance types comprise varying combinations of CPU, memory, storage, and networking capacity and give you the flexibility to choose the appropriate mix of resources for your applications. Each instance type includes one or more instance sizes, allowing you to scale your resources to the requirements of your target workload.

The t2.micro instance type has 1 virtual CPU and 1 GiB of memory.
# Step 4: Key pair (login)
For Key pair name - required,

Amazon EC2 uses public–key cryptography to encrypt and decrypt login information. To ensure you will be able to log in to the guest OS of the instance you create, you identify an existing key pair or create a new key pair when launching the instance. Amazon EC2 then installs the key on the guest OS when the instance is launched.  That way, when you attempt to login to the instance and you provide the private key, you will be authorized to connect to the instance.
# Step 5: Network settings
Next to Network settings, choose Edit.

For VPC, select Lab VPC.

The Lab VPC was created using an AWS CloudFormation template during the setup process of this lab. This VPC includes two public subnets in two different Availability Zones. (you can do so using the console)

Note: Keep the default subnet. This is the subnet in which the instance will run. Notice also that by default, the instance will be assigned a public IP address.

Under Firewall (security groups), choose  Create security group and configure:

Security group name: Web Server security group

Description: Security group for my web server

A security group acts as a virtual firewall that controls the traffic for one or more instances. When you launch an instance, you associate one or more security groups with the instance. You add rules to each security group that allow traffic to or from its associated instances. You can modify the rules for a security group at any time; the new rules are automatically applied to all instances that are associated with the security group.

Under Inbound security group rules, notice that one rule exists. Remove this rule.

# Step 6: Configure storage
In the Configure storage section, keep the default settings.

Amazon EC2 stores data on a network-attached virtual disk called Elastic Block Store.

You will launch the Amazon EC2 instance using a default 8 GiB disk volume. This will be your root volume (also known as a 'boot' volume). 

# Step 7: Advanced details
Expand  Advanced details.

For Termination protection, select Enable.

When an Amazon EC2 instance is no longer required, it can be terminated, which means that the instance is deleted and its resources are released. A terminated instance cannot be accessed again and the data that was on it cannot be recovered. If you want to prevent the instance from being accidentally terminated, you can enable termination protection for the instance, which prevents it from being terminated as long as this setting remains enabled.

Scroll to the bottom of the page and then copy and paste the code shown below into the User data box:

#!/bin/bash
dnf install -y httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
When you launch an instance, you can pass user data to the instance that can be used to perform automated installation and configuration tasks after the instance starts.Your instance is running Amazon Linux 2023. The shell script you have specified will run as the root guest OS user when the instance starts. The script will:

Install an Apache web server (httpd)

Configure the web server to automatically start on boot

Run the Web server once it has finished installing

Create a simple web page
# Step 8: Launch the instance
At the bottom of the Summary panel on the right side of the screen choose Launch instance

You will see a Success message.

Choose View all instances

In the Instances list, select  Web Server. 

Review the information displayed in the Details tab. It includes information about the instance type, security settings and network settings.

The instance is assigned a Public IPv4 DNS that you can use to contact the instance from the Internet.

To view more information, drag the window divider upwards.

At first, the instance will appear in a Pending state, which means it is being launched. It will then change to Initializing, and finally to Running.

Wait for your instance to display the following:

Instance State:  Running

Status Checks:   2/2 checks passed

Congratulations! You have successfully launched your first Amazon EC2 instance.
## Task 2: Monitor Your Instance
Monitoring is an important part of maintaining the reliability, availability, and performance of your Amazon Elastic Compute Cloud (Amazon EC2) instances and your AWS solutions.

Choose the Status checks tab.

With instance status monitoring, you can quickly determine whether Amazon EC2 has detected any problems that might prevent your instances from running applications. Amazon EC2 performs automated checks on every running EC2 instance to identify hardware and software issues.

Notice that both the System reachability and Instance reachability checks have passed.

Choose the Monitoring tab.

This tab displays Amazon CloudWatch metrics for your instance. Currently, there are not many metrics to display because the instance was recently launched.

You can choose the three dots icon in any graph and select Enlarge to see an expanded view of the chosen metric.

Amazon EC2 sends metrics to Amazon CloudWatch for your EC2 instances. Basic (five-minute) monitoring is enabled by default. You can also enable detailed (one-minute) monitoring.

In the Actions  menu towards the top of the console, select Monitor and troubleshoot  Get system log.

The System Log displays the console output of the instance, which is a valuable tool for problem diagnosis. It is especially useful for troubleshooting kernel problems and service configuration issues that could cause an instance to terminate or become unreachable before its SSH daemon can be started. If you do not see a system log, wait a few minutes and then try again.

Scroll through the output and note that the HTTP package was installed from the user data that you added when you created the instance.
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/253f9c84-f1d1-4a9b-9be7-a5cbb51c8838)
Choose Cancel.

Ensure Web Server is still selected. Then, in the Actions  menu, select Monitor and troubleshoot  Get instance screenshot.

This shows you what your Amazon EC2 instance console would look like if a screen were attached to it.
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/3a6a8d38-78ca-4d09-95c0-b7c3edd4e299)
If you are unable to reach your instance via SSH or RDP, you can capture a screenshot of your instance and view it as an image. This provides visibility as to the status of the instance, and allows for quicker troubleshooting.
Choose Cancel.

Congratulations! You have explored several ways to monitor your instance.
# Task 3: Update Your Security Group and Access the Web Server
When you launched the EC2 instance, you provided a script that installed a web server and created a simple web page. In this task, you will access content from the web server. 

Ensure Web Server is still selected.  Choose the Details tab.

Copy the Public IPv4 address of your instance to your clipboard.

Open a new tab in your web browser, paste the IP address you just copied, then press Enter.

Question: Are you able to access your web server? Why not?

You are not currently able to access your web server because the security group is not permitting inbound traffic on port 80, which is used for HTTP web requests. This is a demonstration of using a security group as a firewall to restrict the network traffic that is allowed in and out of an instance.

To correct this, you will now update the security group to permit web traffic on port 80.

Keep the browser tab open, but return to the EC2 Console tab. 

In the left navigation pane, choose Security Groups.

Select  Web Server security group.

Choose the Inbound rules tab.

The security group currently has no inbound rules.

Choose Edit inbound rules, select Add rule and then configure:

Type: HTTP

Source: Anywhere-IPv4

Choose Save rules

Return to the web server tab that you previously opened and refresh  the page.

You should see the message Hello From Your Web Server!

Congratulations! You have successfully modified your security group to permit HTTP traffic into your Amazon EC2 Instance.
# Task 4: Resize Your Instance: Instance Type and EBS Volume
As your needs change, you might find that your instance is over-utilized (too small) or under-utilized (too large). If so, you can change the instance type. For example, if a t2.micro instance is too small for its workload, you can change it to an m5.medium instance. Similarly, you can change the size of a disk.

Stop Your Instance
Before you can resize an instance, you must stop it.

When you stop an instance, it is shut down. There is no runtime charge for a stopped EC2 instance, but the storage charge for attached Amazon EBS volumes remains.

On the EC2 Management Console, in the left navigation pane, choose Instances.

Web Server should already be selected.

In the Instance State  menu, select Stop instance.

Choose Stop

Your instance will perform a normal shutdown and then will stop running.

Wait for the Instance state to display:  Stopped.

Change The Instance Type
In the Actions  menu, select Instance settings  Change instance type, then configure:

Instance Type: t2.small

Choose Apply

When the instance is started again it will run as a t2.small, which has twice as much memory as a t2.micro instance.
# Resize the EBS Volume
With the Web Server instance still selected, choose the Storage tab, select the name of the Volume ID, then select the checkbox next to the volume that displays.

In the Actions  menu, select Modify volume.

The disk volume currently has a size of 8 GiB. You will now increase the size of this disk.

Change the size to: 10 NOTE: You may be restricted from creating large Amazon EBS volumes in this lab.

Choose Modify

Choose Modify again to confirm and increase the size of the volume.
# Start the Resized Instance
You will now start the instance again, which will now have more memory and more disk space.

In left navigation pane, choose Instances.

Select the Web Server instance.

In the Instance state  menu, select Start instance.

Congratulations! You have successfully resized your Amazon EC2 Instance.
# Task 5: Explore EC2 Limits
Amazon EC2 provides different resources that you can use. These resources include images, instances, volumes, and snapshots. When you create an AWS account, there are default limits on these resources on a per-region basis.

In the AWS Management Console, in the search box next to  Services, search for and choose Service Quotas

Choose AWS services from the navigation menu and then in the AWS services Find services search bar, search for ec2 and choose Amazon Elastic Compute Cloud (Amazon EC2).

In the Find quotas search bar, search for running on-demand, but do not make a selection. Instead, observe the filtered list of service quotas that match the criteria.

Notice that there are limits on the number and types of instances that can run in a region. For example, there is a limit on the number of Running On-Demand Standard... instances that you can launch in this region. When launching instances, the request must not cause your usage to exceed the instance limits currently defined in that region.

You can request an increase for many of these limits.
# Task 6: Test Termination Protection
You can delete your instance when you no longer need it. This is referred to as terminating your instance. You cannot connect to or restart an instance after it has been terminated. In this task, you will learn how to use termination protection.

In the AWS Management Console, in the search box next to  Services, search for and choose EC2 to return to the EC2 console.

In left navigation pane, choose Instances.

Select the Web Server instance and in the Instance state  menu, select Terminate instance.

Then choose Terminate

Note that there is a message that says: Failed to terminate the instance i-1234567xxx. The instance 'i-1234567xxx' may not be terminated. Modify its 'disableApiTermination' instance attribute and try again.

This is a safeguard to prevent the accidental termination of an instance. If you really want to terminate the instance, you will need to disable the termination protection.

In the Actions  menu, select Instance settings  Change termination protection. 

Remove the check next to  Enable. 

Choose Save

You can now terminate the instance. 

Select the Web Server instance again and in the Instance state  menu, select Terminate instance.

Choose Terminate
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/cb501430-6b08-4b1a-bcbb-31d62c2cc53f)

Congratulations! You have successfully tested termination protection and terminated your instance.
