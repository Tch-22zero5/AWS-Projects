# Build Your  Amazon Relational Database Service in a Multi-Availability Zone deployment and Interact With Your DB Using an App
Version 4.6.6 (TESS2)
**Amazon Relational Database Service (Amazon RDS)** makes it easy to set up, operate, and scale a relational database in the cloud. It provides cost-efficient and resizable capacity while managing time-consuming database administration tasks, which allows you to focus on your applications and business. Amazon RDS provides you with familiar database engines to choose from: Amazon Aurora MySQL, Amazon Aurora PostgreSQL, Oracle, Microsoft SQL Server, PostgreSQL, MySQL and MariaDB.
# Scenario
You start with the following infrastructure (which was created prior with cloud formation):
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/a71b57fb-b75f-4f32-a511-99b190318da9)

At the end of this project, this is the infrastructure:
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/538e7b4f-12a5-45c8-8392-02e1c6521460)
# Task 1: Create a Security Group for the RDS DB Instance
In this task, you will create a security group to allow your web server to access your RDS DB instance. The security group will be used when you launch the database instance.

In the AWS Management Console, on the Services  menu, choose VPC.

In the left navigation pane, choose Security Groups.

Choose Create security group and then configure:

Security group name: DB Security Group
Description: Permit access from Web Security Group
VPC: Choose your VPC (for this project I used Lab VPC)
You will now add a rule to the security group to permit inbound database requests.

In the Inbound rules pane, choose Add rule

The security group currently has no rules. You will add a rule to permit access from the Web Security Group.

Configure the following settings:

Type: MySQL/Aurora (3306)
CIDR, IP, Security Group or Prefix List: Type sg and then select Web Security Group.
This configures the Database security group to permit inbound traffic on port 3306 from any EC2 instance that is associated with the Web Security Group.

Choose Create security group

You will use this security group when launching the Amazon RDS database.
# Task 2: Create a DB Subnet Group
In this task, you will create a DB subnet group that is used to tell RDS which subnets can be used for the database. Each DB subnet group requires subnets in at least two Availability Zones.

On the Services  menu, choose RDS.

In the left navigation pane, choose Subnet groups.

If the navigation pane is not visible, choose the  menu icon in the top-left corner.

Choose Create DB Subnet Group then configure:

Name: DB-Subnet-Group
Description: DB Subnet Group
VPC: Choose your VPC (for this project I used Lab VPC)
Scroll down to the Add Subnets section.

Expand the list of values under Availability Zones and  select the first two zones: us-east-1a and us-east-1b.

Expand the list of values under Subnets and select the subnets associated with the CIDR ranges 10.0.1.0/24 and 10.0.3.0/24.

These subnets should now be shown in the Subnets selected  table.

Choose Create

You will use this DB subnet group when creating the database in the next task.
# Task 3: Create an Amazon RDS DB Instance
In this task, you will configure and launch a Multi-AZ Amazon RDS for MySQL database instance.

Amazon RDS Multi-AZ deployments provide enhanced availability and durability for Database (DB) instances, making them a natural fit for production database workloads. When you provision a Multi-AZ DB instance, Amazon RDS automatically creates a primary DB instance and synchronously replicates the data to a standby instance in a different Availability Zone (AZ).

In the left navigation pane, choose Databases.

Choose Create database

If you see Switch to the new database creation flow at the top of the screen, please choose it.

Select  MySQL under Engine Options.

Under Templates choose Dev/Test.

Under Availability and durability choose Multi-AZ DB instance.

Under Settings, configure:

DB instance identifier: Choose a name ( for this project i chose lab-db)
Master username: main
Master password: lab-password
Confirm password: lab-password
Under DB instance class, configure:

Select  Burstable classes (includes t classes).
Select db.t3.micro
Under Storage, configure:

Storage type: General Purpose (SSD)
Allocated storage: 20
Under Connectivity, configure:

Virtual Private Cloud (VPC): Lab VPC
Under Existing VPC security groups, from the dropdown list:

Choose DB Security Group.
Deselect default.
Under Monitoring expand Additional configuration.

Uncheck Enable Enhanced monitoring.
Under Additional configuration, configure:

Initial database name: lab
Uncheck Enable automatic backups.
Uncheck Enable encryption
This will turn off backups, which is not normally recommended, but will make the database deploy faster for this lab.

Choose Create database

Your database will now be launched.

If you receive an error that mentions "not authorized to perform: iam:CreateRole", make sure you unchecked Enable Enhanced monitoring in the previous step.

Choose lab-db (choose the link itself).

You will now need to wait approximately 4 minutes for the database to be available. The deployment process is deploying a database in two different Availability zones.

While you are waiting, you might want to review the Amazon RDS FAQs or grab a cup of coffee.

Wait until Info changes to Modifying or Available.

Scroll down to the Connectivity & security section and copy the Endpoint field.

It will look similar to: lab-db.cggq8lhnxvnv.us-west-2.rds.amazonaws.com

Paste the Endpoint value into a text editor. You will use it later.
# Task 4: Interact with Your Database
In this task, you will open a web application running on your web server and configure it to use the database.

To copy the WebServer IP address, choose on the Details drop down menu above these instructions, and then choose Show.

Open a new web browser tab, paste the WebServer IP address and press Enter.

The web application will be displayed, showing information about the EC2 instance.

Choose the RDS link at the top of the page.

You will now configure the application to connect to your database.

Configure the following settings:

Endpoint: Paste the Endpoint you copied to a text editor earlier
Database: lab
Username: main
Password: lab-password
Choose Submit
A message will appear explaining that the application is running a command to copy information to the database. After a few seconds the application will display an Address Book.

The Address Book application is using the RDS database to store information.

Test the web application by adding, editing and removing contacts.

The data is being persisted to the database and is automatically replicating to the second Availability Zone.
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/538e7b4f-12a5-45c8-8392-02e1c6521460)
Congratulations! You have completed this infrastructure.
# Attributions
Bootstrap v3.3.5 - http://getbootstrap.com/

The MIT License (MIT)
Copyright (c) 2011-2016 Twitter, Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
