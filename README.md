# Wordpress-Evolution
Launch the CloudFormation template for the WP VPC to create the VPC that WordPress will run on.
Create an EC2 instance with Amazon Linux 2 AMI, instance type of t2.micro, and 8 GiB gp2 root volume in the newly created VPC.

Create new parameters in the Parameter Store. Create a Parameter for DBUser, DBName, DBEndpoint, DBPassword, & DBRootPassword.

Connect to Session Manager for the EC2 instance you just created. Run the commands in the Parameters_Values text to bring the parameter store values into the enviorment.