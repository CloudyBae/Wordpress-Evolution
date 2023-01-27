# Wordpress-Evolution
# MANUAL WORDPRESS INSTALLATION ON SINGLE INSTANCE
Launch the CloudFormation template for the WP VPC to create the VPC that WordPress will run on.
Create an EC2 instance with Amazon Linux 2 AMI, instance type of t2.micro, and 8 GiB gp2 root volume in the newly created VPC.
![Screenshot_1](https://user-images.githubusercontent.com/109190196/215160032-e35f1e6b-d0bf-481e-8323-4193f5390326.jpg)
Create new parameters in the Parameter Store. Create a Parameter for DBUser, DBName, DBEndpoint, DBPassword, & DBRootPassword.
![Screenshot_2](https://user-images.githubusercontent.com/109190196/215160073-f046ba24-aae0-4c2d-8891-8bbe160142dd.jpg)
Connect to Session Manager for the EC2 instance you just created. Run the commands in the Parameters_Values text to bring the parameter store values into the enviorment. Update & upgrade the packages. Install the MariaDB server, Apache webserver, wget some libraries, and stress test utilities. Run commands to enable automatic startup when the instance in running. Set MariaDB Root Password. Download & extract Wordpress. Configure wp-config.php file. Fix perms on the filesystem. Create WP User & database. 
![Screenshot_3](https://user-images.githubusercontent.com/109190196/215160122-64c5f926-274c-480d-a6f6-c432da9e5263.jpg)
Use the EC2 public ipv4 and open the Wordpress Webserver. Create a site title, username, password, email, and install WordPress. Create a WordPress post to test that it is working.
![Screenshot_4](https://user-images.githubusercontent.com/109190196/215160161-9c7245c2-3011-43c5-876e-4fdb0eaa77b9.jpg)
![Screenshot_5](https://user-images.githubusercontent.com/109190196/215160176-da240148-6723-4042-8547-21dd783ce43e.jpg)

# Set up same Wordpress installation as above through a Launch Template (Automate the build with a Launch Template) 
Create a new launch template to auotmate the installation of everything we did in Session Manager above. Use the Amazon Linux 2 AMI SSD Volume Type and t2.micro instance type. Paste the contents in User_Data_LT into the User Data to bootstrap. Launch instance from the newly created template.
![Screenshot_6](https://user-images.githubusercontent.com/109190196/215160223-d2fbfbe6-d3a2-4fd4-a0f1-c8a8e37e4794.jpg)
![Screenshot_8](https://user-images.githubusercontent.com/109190196/215160385-3418a6cf-00de-4a2e-a08b-db684ae67bb2.jpg)
Use the EC2 public ipv4 and open the Wordpress Webserver. Create a site title, username, password, email, and install WordPress. Create a WordPress post to test that it is working.
![Screenshot_9](https://user-images.githubusercontent.com/109190196/215160412-d2b06c28-0d03-4893-904d-16437fb729e6.jpg)

# Migrate the database in the EC2 instance into a separate RDS instance (DB now lives past the lifetime of an EC2 instance & can grow or shrink DB independently)
Create RDS Subnet groups. Create RDS Database with a MySQL Engine. Open session manager of the EC2 instance to migrate WordPress data from MariaDB to RDS. Populate variables with the data from the Parameter store. Take a backup of the local DB. Delete the DBEndpoint parameter in Parameter and make a new DBEndpoint paramater with the value pointing at the RDS endpoint. Go back to session manager and update the DBEndpoint. Restore the database export into RDS. Update WordPress config file to point at RDS. Disable MariaDB.
![Screenshot_10](https://user-images.githubusercontent.com/109190196/215160629-77d961c3-c858-48bf-aaea-b0b31253d294.jpg)
![Screenshot_11](https://user-images.githubusercontent.com/109190196/215160637-ff35fb15-02e8-45e3-a658-5aba073c7cf6.jpg)
![Screenshot_12](https://user-images.githubusercontent.com/109190196/215160644-708e0316-e60d-4d91-bc32-902adf89452e.jpg)
We can see this database migration was successful by opening the EC2 public ipv4 and seeing the same WordPress post.
![Screenshot_13](https://user-images.githubusercontent.com/109190196/215160662-c6fd5aae-b77a-4825-aebb-cf0139377b5e.jpg)
Update the Launch Template to use this new configuration of using RDS. In the User Data remove the commands to enable and start MariaDB, delete the command that sets a password to MariaDB, and delete the commands to create a MariaDB user. Set the default version for the Launch Template to use version 2.
![Screenshot_14](https://user-images.githubusercontent.com/109190196/215160700-c5a034fc-0b15-433c-b745-cf90831ee32d.jpg)

# Migrate the WP locally stored media into an EFS file system (Data can now be used across all instances and live past the lifetime of the instance)
Create a new EFS file system with general purpose performance and bursting throughput mode. Configure mount targets.
![Screenshot_15](https://user-images.githubusercontent.com/109190196/215160791-e171504c-43e8-43f1-a422-6cbc1ef3f5e0.jpg)
Create a new parameter in the Parameter Store to point at the EFS file system you just created.
![Screenshot_16](https://user-images.githubusercontent.com/109190196/215160822-0032d9c8-94c6-48e8-b963-493dacec1573.jpg)
Open Session Manager in the EC2 instance. Install the amazon EFS utilities. Move the wp-content folder to a temporary folder. Create a new empty directory. Get the EFS file system ID from the parameter store. Configure the EFS file system to mount in the new directory you just created. Migrate the data in the temporary folder back in to the wp-content folder in the EFS. Reboot. Let the instance reboot and connect into session manager again. If migration was successful you will see the EFS file system is still mounted into the directory that was created. You can also open the browser with the EC2 public ipv4 and the same WordPress post should load up if successful. 
![Screenshot_17](https://user-images.githubusercontent.com/109190196/215160859-227794e4-3866-406d-ae94-8076576c7277.jpg)
![Screenshot_18](https://user-images.githubusercontent.com/109190196/215160874-85af87bb-0e7e-4007-81a9-b705f082a559.jpg)
Update the Launch Template. In the user data of the launch template, update it so that it uses EFS now. Make this new version the default.
![Screenshot_19](https://user-images.githubusercontent.com/109190196/215160912-170113e8-4efe-4853-b622-d64ab81b419f.jpg)

# Add an auto scaling group to provision and terminate instances automatically. Add a load balancer to abstract connection away from individual instance to allow elastic scaling and self healing.
Create an application load balancer. Create a new target group, use this target group for the ALB we are creating. 
![Screenshot_20](https://user-images.githubusercontent.com/109190196/215160944-7535681a-9727-4f89-abd0-c32d776358b4.jpg)
Create a new parameter in the Parameter Store to point at the ALB.
![Screenshot_21](https://user-images.githubusercontent.com/109190196/215160976-df7cd733-c717-4d72-9181-af131fc8e81f.jpg)
Update the Launch Template. Update the User data to use the ALB. Set default version to use the latest version.
![Screenshot_22](https://user-images.githubusercontent.com/109190196/215160999-07e9f62a-0df4-486c-9b42-f4dd93bb4681.jpg)
Create an Auto Scaling Group using the lateast launch template. Attach the ASG to the ALB and use ELB health checks. Enable group metrics collection within CloudWatch.
![Screenshot_23](https://user-images.githubusercontent.com/109190196/215161027-a7671814-eb25-482a-b5a7-2bacb2b29843.jpg)
Terminate the running EC2 instance and if successfully configured, ASG will provision a new instance to maintain the desired state amount we set.
![Screenshot_25](https://user-images.githubusercontent.com/109190196/215161185-abe7cde0-6936-4af9-b881-a364247b457f.jpg)
![Screenshot_24](https://user-images.githubusercontent.com/109190196/215161120-386b0031-4714-4baa-85a7-4af21ef30fe6.jpg)
Create a dynamic scaling policy for the auto scaling group. Create 2 policies, scale in and scale out. Scale out when CPU usage on average is above 40%. Scale in when CPU usage on average is below 40%. Create a CloudWatch Alarm for both policies.
![Screenshot_26](https://user-images.githubusercontent.com/109190196/215161243-f309785d-d9da-4c2a-aa99-d96b787d030b.jpg)
![Screenshot_27](https://user-images.githubusercontent.com/109190196/215161252-cc71280b-1d69-4b1c-a824-d1e5b1714474.jpg)
Test out the ASG but simluating load. Go into the Session Manager and add stress. If everything is successfully configured, the ASG will kick in and provision an instance to the increased load.
![Screenshot_28](https://user-images.githubusercontent.com/109190196/215161269-6e6b1709-406d-4689-9703-8b2029fb17d8.jpg)
![Screenshot_29](https://user-images.githubusercontent.com/109190196/215161315-0dc3cac8-b34a-44d8-8cdb-54f7904cf209.jpg)
![Screenshot_30](https://user-images.githubusercontent.com/109190196/215161320-b1d2ce19-f087-44ed-a159-f29ec9f316ef.jpg)
If an instance gets terminated or fails a health check, another instance will be provisioned to provide self healing.
![Screenshot_31](https://user-images.githubusercontent.com/109190196/215161362-92a7ab04-7c21-49e8-aad9-d72e4e6dc4ab.jpg)
Use the ALB DNS name to open the WordPress application in your browser and if the same WordPress post loads then this architecture evolution is complete. This WordPress application is now a fully scalable, self healing, resilient architecture.
![Screenshot_32](https://user-images.githubusercontent.com/109190196/215161396-2a19c562-24b0-4d40-894b-7ad7cfa8dfe5.jpg)
