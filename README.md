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

# Migrate the database in the EC2 instance into a separate RDS instance (Data in DB now lives past the life of an EC2 instance & can grow or shrink DB independently)
Create RDS Subnet groups. Create RDS Database with a MySQL Engine. Open session manager of the EC2 instance to migrate WordPress data from MariaDB to RDS. Populate variables with the data from the Parameter store. Take a backup of the local DB. Delete the DBEndpoint parameter in Parameter and make a new DBEndpoint paramater with the value pointing at the RDS endpoint. Go back to session manager and update the DBEndpoint. Restore the database export into RDS. Update WordPress config file to point at RDS. Disable MariaDB.

We can see this database migration was successful by opening the EC2 public ipv4 and seeing the same WordPress post.

Update the Launch Template to use this new configuration of using RDS. In the User Data remove the commands to enable and start MariaDB, delete the command that sets a password to MariaDB, and delete the commands to create a MariaDB user. Set the default version for the Launch Template to use version 2.