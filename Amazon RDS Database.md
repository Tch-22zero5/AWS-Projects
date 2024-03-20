# Creating an Amazon RDS Database
# overview
Traditionally, creating a database can be a complex process that requires either a database administrator or a systems administrator. In the cloud, you can simplify this process by using Amazon Relational Database Service (Amazon RDS).
# objectives
Launch a database using Amazon RDS 

Configure a web application to connect to the database instance
# Task 1: Creating an Amazon RDS database
In this task, you will create a MySQL database in your virtual private cloud (VPC). MySQL is a popular open source relational database management system (RDBMS), so there are no software licensing fees.
**Windows Users:** Use Chrome or Firefox as your web browser for this lab. The lab instructions are not compatible with Internet Explorer because of a difference in the Amazon RDS console.
 
In the search box to the right of Services, search for and choose RDS to open the RDS console.
 
Choose Create database
 
Under Engine options, select MySQL.

The options include several use cases, ranging from enterprise-class databases to Dev/Test systems. In the options, you might notice Amazon Aurora. Aurora is a MySQL-compatible system that was re-architected for the cloud. If your company uses large-scale MySQL or PostgreSQL databases, Aurora can provide enhanced performance.
 
Set the templates and availability and durability options:

Under the Templates section, select Dev/Test.

Under the Availability and durability section, select Single DB instance 

Note: the default Multi-AZ deployment option automatically creates a replica of the database in a second Availability Zone for High Availability, however that is not needed.
 
Under the Settings section, configure these options:

DB instance identifier: inventory-db

Username: admin

Password: my-password or lab-password

Confirm password:  my-password or lab-password
 
Under the DB instance class section, configure these options:

Select Burstable classes (includes t classes).

Select db.t3.micro
 
Under the Connectivity section, configure these options: 

Virtual Private Cloud (VPC): Your VPC or Lab VPC

Existing VPC security groups: 

Choose DB-SG. It will be highlighted. 

Remove the default security group.
 
Expand the Additional configuration panel, then configure these settings:

Initial database name: inventory

Note: This is the logical name of the database that will be used by the application.

Clear (turn off) the Enable Enhanced monitoring option
 
Feel free to review the many other options displayed on the page, but leave them set to their default values. Options include automatic backups, the ability to export log files, and automatic version upgrades. The ability to activate these features through check boxes demonstrates the power of using a fully managed database solution instead of installing, backing up, and maintaining the database yourself.
 
Choose Create database (at the bottom of the page).

You should receive a message indicating that your database is being created.

If you receive an error message that mentions rds-monitoring-role, confirm that you have cleared (turned off) the Enhanced Monitoring option in the previous step, then try again.

Before you continue to the next task, the database instance status must be Available. This process might take several minutes.
# Task 2: Configuring web application communication with a database instance
# I have already automatically deployed an Amazon Elastic Compute Cloud (Amazon EC2) instance  named App Server with a running web application. **DO THE SAME** You must use the IP address of the instance to connect to the application.
 
In the search box to the right of Services, search for and choose EC2 to open the EC2 console.
 
In the left navigation pane, choose Instances.

In the center pane, there should be a running instance that is named App Server.
 
Select the App Server instance.
 
In the Details tab, copy the Public IPv4 address to your clipboard.

Tip: If you hover over the IP address, a copy icon appears. To copy the displayed value, choose the icon.
 
Open a new web browser tab, paste the IP address into the address bar, and then press ENTER.

The web application should appear. It does not display much information because the application is not yet connected to the database.
 
Choose Settings.

You can now configure the application to use the RDS DB instance you created earlier. You will first retrieve the Database Endpoint so that the application knows how to connect to a database.
 
Return to the AWS Management Console, but do not close the application tab. (You will return to it soon).
 
In the Services search box, search for and choose RDS to open the RDS console.
 
In the left navigation pane, choose Databases.
 
Choose inventory-db.
 
Scroll to the Connectivity & Security section and copy the Endpoint to your clipboard.

It should look similar to this example: inventory-db.crwxbgqad61a.rds.amazonaws.com
 
Return to the browser tab with the Inventory application, and enter these values:

Endpoint: Paste the endpoint you copied earlier

Database: inventory

Username: admin

Password: lab-password

Choose Save

The application will now connect to the database, load some initial data, and display information.
 
Add inventory, edit, and delete inventory information by using the web application.

The inventory information is stored in the Amazon RDS MySQL database that you created earlier in the lab. This means that any failure in the application server will not lose any data. It also means that multiple application servers can access the same data.
 
Insert new records into the table. Ensure that the table has 5 or more inventory records.

You have now successfully launched the application and connected it to the database!

# Optional: You can access the saved parameters in the Systems Manager console, under Parameter Store.
