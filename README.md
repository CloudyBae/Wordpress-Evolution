# Wordpress-Evolution
# MANUAL WORDPRESS INSTALLATION ON SINGLE INSTANCE
Launch the CloudFormation template for the WP VPC to create the VPC that WordPress will run on.
Create an EC2 instance with Amazon Linux 2 AMI, instance type of t2.micro, and 8 GiB gp2 root volume in the newly created VPC.

Create new parameters in the Parameter Store. Create a Parameter for DBUser, DBName, DBEndpoint, DBPassword, & DBRootPassword.

Connect to Session Manager for the EC2 instance you just created. Run the commands in the Parameters_Values text to bring the parameter store values into the enviorment. Update & upgrade the packages. Install the MariaDB server, Apache webserver, wget some libraries, and stress test utilities. Run commands to enable automatic startup when the instance in running. Set MariaDB Root Password. Download & extract Wordpress. Configure wp-config.php file. Fix perms on the filesystem. Create WP User & database. 

Use the EC2 public ipv4 and open the Wordpress Webserver. Create a site title, username, password, email, and install WordPress. Create a WordPress post to test that it is working.

# Set up same Wordpress installation as above through a Launch Template (Automate the build with a Launch Template) 
Create a new launch template to auotmate the installation of everything we did in Session Manager above. Use the Amazon Linux 2 AMI SSD Volume Type and t2.micro instance type. Paste the contents in User_Data_LT into the User Data to bootstrap. Launch instance from the newly created template.

Use the EC2 public ipv4 and open the Wordpress Webserver. Create a site title, username, password, email, and install WordPress. Create a WordPress post to test that it is working.

# Migrate the database in the EC2 instance into a separate RDS instance (DB now lives past the lifetime of an EC2 instance & can grow or shrink DB independently)
Create RDS Subnet groups. Create RDS Database with a MySQL Engine. Open session manager of the EC2 instance to migrate WordPress data from MariaDB to RDS. Populate variables with the data from the Parameter store. Take a backup of the local DB. Delete the DBEndpoint parameter in Parameter and make a new DBEndpoint paramater with the value pointing at the RDS endpoint. Go back to session manager and update the DBEndpoint. Restore the database export into RDS. Update WordPress config file to point at RDS. Disable MariaDB.

We can see this database migration was successful by opening the EC2 public ipv4 and seeing the same WordPress post.

Update the Launch Template to use this new configuration of using RDS. In the User Data remove the commands to enable and start MariaDB, delete the command that sets a password to MariaDB, and delete the commands to create a MariaDB user. Set the default version for the Launch Template to use version 2.

# Migrate the WP locally stored media into an EFS file system (Data can now be used across all instances and live past the lifetime of the instance)
Create a new EFS file system with general purpose performance and bursting throughput mode. Configure mount targets.

Create a new parameter in the Parameter Store to point at the EFS file system you just created.

Open Session Manager in the EC2 instance. Install the amazon EFS utilities. Move the wp-content folder to a temporary folder. Create a new empty directory. Get the EFS file system ID from the parameter store. Configure the EFS file system to mount in the new directory you just created. Migrate the data in the temporary folder back in to the wp-content folder in the EFS. Reboot. Let the instance reboot and connect into session manager again. If migration was successful you will see the EFS file system is still mounted into the directory that was created. You can also open the browser with the EC2 public ipv4 and the same WordPress post should load up if successful. 

Update the Launch Template. In the user data of the launch template, update it so that it uses EFS now. Make this new version the default.

# Add an auto scaling group to provision and terminate instances automatically. Add a load balancer to abstract connection away from individual instance to allow elastic scaling and self healing.
Create an application load balancer. Create a new target group, use this target group for the ALB we are creating. 

Create a new parameter in the Parameter Store to point at the ALB.

Update the Launch Template. Update the User data to use the ALB. Set default version to use the latest version.

Create an Auto Scaling Group using the lateast launch template. Attach the ASG to the ALB and use ELB health checks. Enable group metrics collection within CloudWatch.

Terminate the running EC2 instance and if successfully configured, ASG will provision a new instance to maintain the desired state amount we set.

Create a dynamic scaling policy for the auto scaling group. Create 2 policies, scale in and scale out. Scale out when CPU usage on average is above 40%. Scale in when CPU usage on average is below 40%. Create a CloudWatch Alarm for both policies.

Test out the ASG but simluating load. Go into the Session Manager and add stress. If everything is successfully configured, the ASG will kick in and provision an instance to the increased load.

If an instance gets terminated or fails a health check, another instance will be provisioned to provide self healing.

Use the ALB DNS name to open the WordPress application in your browser and if the same WordPress post loads then this architecture evolution is complete. This WordPress application is now a fully scalable, self healing, resilient architecture.