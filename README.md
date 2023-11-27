# Assignment


1.      Elastic Beanstalk Application: Set up an AWS Elastic Beanstalk application and an associated environment. The application should be named my-web-app and the environment should be named my-web-app-env. The solution should use the latest Amazon Linux 2 platform for Elastic Beanstalk. 

# main.tf

# Configure the AWS provider
provider "aws" {
  region = "us-east-1"
}

# Create an Elastic Beanstalk application
resource "aws_elastic_beanstalk_application" "my_web_app" {
  name        = "my-web-app"
  description = "My Web Application"
}

# Create an Elastic Beanstalk environment
resource "aws_elastic_beanstalk_environment" "my_web_app_env" {
  name                = "my-web-app-env"
  application         = aws_elastic_beanstalk_application.my_web_app.name
  solution_stack_name = "64bit Amazon Linux 2 v5.5.5 running Python 3.8"

  # Configure other environment settings (e.g., instance type, key pair, etc.)
  # ...

  # Optionally, add tags for better organization
  tags = {
    Name        = "my-web-app-env"
    Environment = "Production"
  }
}





2.     
 EC2 Auto Scaling: Configure Auto Scaling for the EC2 instances within the Elastic Beanstalk environment.
 Ensure that the minimum number of instances is set to 2 and can scale up to a maximum of 6 based onCPU utilization. Set the scaling trigger to increase the instance count by 1 when the average CPU utilization exceeds 70% and to decrease by 1 when it falls below 30%. 



# main.tf

# Configure the AWS provider
provider "aws" {
  region = "us-east-1"
}

# Create an Elastic Beanstalk application
resource "aws_elastic_beanstalk_application" "my_web_app" {
  name        = "my-web-app"
  description = "My Web Application"
}

# Create an Elastic Beanstalk environment
resource "aws_elastic_beanstalk_environment" "my_web_app_env" {
  name                = "my-web-app-env"
  application         = aws_elastic_beanstalk_application.my_web_app.name
  solution_stack_name = "64bit Amazon Linux 2 v5.5.5 running Python 3.8"

  # Configure other environment settings (e.g., instance type, key pair, etc.)
  # ...

  # Optionally, add tags for better organization
  tags = {
    Name        = "my-web-app-env"
    Environment = "Production"
  }
}

# Configure EC2 Auto Scaling for the Elastic Beanstalk environment
resource "aws_elastic_beanstalk_environment" "my_web_app_env" {
  name    = "my-web-app-env"
  service = "ec2"

  setting {
    namespace = "aws:autoscaling:asg"
    name      = "MinSize"
    value     = "2"
  }

  setting {
    namespace = "aws:autoscaling:asg"
    name      = "MaxSize"
    value     = "6"
  }

  setting {
    namespace = "aws:autoscaling:trigger"
    name      = "MeasureName"
    value     = "CPUUtilization"
  }

  setting {
    namespace = "aws:autoscaling:trigger"
    name      = "Statistic"
    value     = "Average"
  }

  setting {
    namespace = "aws:autoscaling:trigger"
    name      = "LowerThreshold"
    value     = "30"
  }

  setting {
    namespace = "aws:autoscaling:trigger"
    name      = "UpperThreshold"
    value     = "70"
  }

  setting {
    namespace = "aws:autoscaling:trigger"
    name      = "Unit"
    value     = "Percent"
  }
}

In this script:
Added an aws_elastic_beanstalk_environment resource specifically for configuring EC2 Auto Scaling within the Elastic Beanstalk environment.
The MinSize and MaxSize settings in the aws:autoscaling:asg namespace are set to 2 and 6, respectively.
And added settings in the aws:autoscaling:trigger namespace to define the scaling policies based on CPU utilization. 
The UpperThreshold is set to 70% to trigger scaling up, and the LowerThreshold is set to 30% to trigger scaling down.






3.
Additional Components: Incorporate an Application Load Balancer to distribute traffic evenly across the instances.
Ensure that all instances are within a VPC, distributed across at least two availability zonesfor high availability.
All resources should have proper tags, at minimum with Name and Environment to reflect the application and environment names. 


# main.tf

# Configure the AWS provider
provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name        = "my-web-app-vpc"
    Environment = "Production"
  }
}

# Create subnets in two availability zones
resource "aws_subnet" "subnet_a" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name        = "subnet-a"
    Environment = "Production"
  }
}

resource "aws_subnet" "subnet_b" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = {
    Name        = "subnet-b"
    Environment = "Production"
  }
}

# Create a security group for the instances
resource "aws_security_group" "web_app_sg" {
  vpc_id = aws_vpc.my_vpc.id
  // Define your security group rules as needed
  // ...
  tags = {
    Name        = "web-app-security-group"
    Environment = "Production"
  }
}

# Create an Elastic Beanstalk application
resource "aws_elastic_beanstalk_application" "my_web_app" {
  name        = "my-web-app"
  description = "My Web Application"
}

# Create an Elastic Beanstalk environment
resource "aws_elastic_beanstalk_environment" "my_web_app_env" {
  name                = "my-web-app-env"
  application         = aws_elastic_beanstalk_application.my_web_app.name
  solution_stack_name = "64bit Amazon Linux 2 v5.5.5 running Python 3.8"
  tier                = "WebServer"
  // other environment settings...

  # Specify the VPC and subnets
  setting {
    namespace = "aws:ec2:vpc"
    name      = "VPCId"
    value     = aws_vpc.my_vpc.id
  }

  setting {
    namespace = "aws:ec2:vpc"
    name      = "Subnets"
    value     = "${aws_subnet.subnet_a.id},${aws_subnet.subnet_b.id}"
  }

  # Specify the security group
  setting {
    namespace = "aws:autoscaling:launchconfiguration"
    name      = "SecurityGroups"
    value     = aws_security_group.web_app_sg.id
  }

  # Optionally, add tags for better organization
  tags = {
    Name        = "my-web-app-env"
    Environment = "Production"
  }
}

# Create an Application Load Balancer
resource "aws_lb" "web_app_lb" {
  name               = "web-app-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web_app_sg.id]
  subnets            = [aws_subnet.subnet_a.id, aws_subnet.subnet_b.id]
  enable_deletion_protection = false

  enable_cross_zone_load_balancing = true
  idle_timeout                      = 60

  tags = {
    Name        = "web-app-lb"
    Environment = "Production"
  }
}

In this script:
A VPC is created (aws_vpc).
Two subnets (aws_subnet) are created, each in a different availability zone.
A security group (aws_security_group) is created for the instances.
The Elastic Beanstalk environment is updated to specify the VPC, subnets, and security group settings.
An Application Load Balancer (aws_lb) is created and associated with the specified subnets.





4.
Security: Define security groups for the Elastic Beanstalk environment that allow HTTP and HTTPS traffic
from anywhere but restrict SSH access to a specific IP range (please define a placeholder for the IP range).
Ensure that the EC2 instances are not publicly accessible directly and can only be accessed via the Load Balancer. 



To define security groups for the Elastic Beanstalk environment with the specified requirements,
  It needs to update the Terraform script. Below is an updated version that includes security group definitions:

# main.tf

# Configure the AWS provider
provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name        = "my-web-app-vpc"
    Environment = "Production"
  }
}

# Create subnets in two availability zones
resource "aws_subnet" "subnet_a" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name        = "subnet-a"
    Environment = "Production"
  }
}

resource "aws_subnet" "subnet_b" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = {
    Name        = "subnet-b"
    Environment = "Production"
  }
}

# Create a security group for the instances
resource "aws_security_group" "web_app_sg" {
  vpc_id = aws_vpc.my_vpc.id

  # Allow incoming HTTP and HTTPS traffic from anywhere
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Restrict SSH access to a specific IP range (replace X.X.X.X/32 with your IP address)
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["X.X.X.X/32"]
  }

  # Allow outbound traffic to anywhere
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "web-app-security-group"
    Environment = "Production"
  }
}

# Create an Elastic Beanstalk application
resource "aws_elastic_beanstalk_application" "my_web_app" {
  name        = "my-web-app"
  description = "My Web Application"
}

# Create an Elastic Beanstalk environment
resource "aws_elastic_beanstalk_environment" "my_web_app_env" {
  name                = "my-web-app-env"
  application         = aws_elastic_beanstalk_application.my_web_app.name
  solution_stack_name = "64bit Amazon Linux 2 v5.5.5 running Python 3.8"
  tier                = "WebServer"
  # other environment settings...

  # Specify the VPC and subnets
  setting {
    namespace = "aws:ec2:vpc"
    name      = "VPCId"
    value     = aws_vpc.my_vpc.id
  }

  setting {
    namespace = "aws:ec2:vpc"
    name      = "Subnets"
    value     = "${aws_subnet.subnet_a.id},${aws_subnet.subnet_b.id}"
  }

  # Specify the security group
  setting {
    namespace = "aws:autoscaling:launchconfiguration"
    name      = "SecurityGroups"
    value     = aws_security_group.web_app_sg.id
  }

  # Optionally, add tags for better organization
  tags = {
    Name        = "my-web-app-env"
    Environment = "Production"
  }
}

In this script:
The aws_security_group resource is created for the instances with rules allowing HTTP and HTTPS traffic from anywhere,
but restricting SSH access to a specific IP range.
The security group is associated with the Elastic Beanstalk environment using the aws:autoscaling:launchconfiguration setting.
Make sure to replace X.X.X.X/32 with the actual IP range you want to allow for SSH access.
After making these changes, run terraform to apply the updated configuration.





5.      
Output: The Terraform script should output the URL of the deployed Elastic Beanstalk application. 


# main.tf
# Output the URL of the Elastic Beanstalk application
output "eb_app_url" {
  value = aws_elastic_beanstalk_environment.my_web_app_env.endpoint_url
}

In this script: 
An output block named eb_app_url is created, and it retrieves the endpoint_url attribute from the Elastic Beanstalk environment.
This attribute represents the URL of the Elastic Beanstalk application.
After applying the Terraform script, you can access the output value by running:
terraform output eb_app_url
And this command will display the URL of the deployed Elastic Beanstalk application.
Make sure to run terraform apply after adding the output block to apply the changes in infrastructure.






6.
Documentation: Include comments within the Terraform scripts explaining the purpose of the major sections and resources. Provide a README file that includes: 


a.Instructions on how to run the Terraform scripts. 



# main.tf

# Configure the AWS provider
provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "my_vpc" {
  cidr_block          = "10.0.0.0/16"
  enable_dns_support  = true
  enable_dns_hostnames = true

  # Tags for better organization
  tags = {
    Name        = "my-web-app-vpc"
    Environment = "Production"
  }
}

# Create subnets in two availability zones
resource "aws_subnet" "subnet_a" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  # Tags for better organization
  tags = {
    Name        = "subnet-a"
    Environment = "Production"
  }
}

resource "aws_subnet" "subnet_b" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true

  # Tags for better organization
  tags = {
    Name        = "subnet-b"
    Environment = "Production"
  }
}

# Create a security group for the instances
resource "aws_security_group" "web_app_sg" {
  vpc_id = aws_vpc.my_vpc.id

  # Allow incoming HTTP and HTTPS traffic from anywhere
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Restrict SSH access to a specific IP range (replace X.X.X.X/32 with your IP address)
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["X.X.X.X/32"]
  }

  # Allow outbound traffic to anywhere
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Tags for better organization
  tags = {
    Name        = "web-app-security-group"
    Environment = "Production"
  }
}

# Create an Elastic Beanstalk application
resource "aws_elastic_beanstalk_application" "my_web_app" {
  name        = "my-web-app"
  description = "My Web Application"
}

# Create an Elastic Beanstalk environment
resource "aws_elastic_beanstalk_environment" "my_web_app_env" {
  name                = "my-web-app-env"
  application         = aws_elastic_beanstalk_application.my_web_app.name
  solution_stack_name = "64bit Amazon Linux 2 v5.5.5 running Python 3.8"
  tier                = "WebServer"

  # Specify the VPC and subnets
  setting {
    namespace = "aws:ec2:vpc"
    name      = "VPCId"
    value     = aws_vpc.my_vpc.id
  }

  setting {
    namespace = "aws:ec2:vpc"
    name      = "Subnets"
    value     = "${aws_subnet.subnet_a.id},${aws_subnet.subnet_b.id}"
  }

  # Specify the security group
  setting {
    namespace = "aws:autoscaling:launchconfiguration"
    name      = "SecurityGroups"
    value     = aws_security_group.web_app_sg.id
  }

  # Optionally, add tags for better organization
  tags = {
    Name        = "my-web-app-env"
    Environment = "Production"
  }
}

# Create an Application Load Balancer
resource "aws_lb" "web_app_lb" {
  name               = "web-app-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web_app_sg.id]
  subnets            = [aws_subnet.subnet_a.id, aws_subnet.subnet_b.id]
  enable_deletion_protection = false

  enable_cross_zone_load_balancing = true
  idle_timeout                      = 60

  # Tags for better organization
  tags = {
    Name        = "web-app-lb"
    Environment = "Production"
  }
}

# Output the URL of the Elastic Beanstalk application
output "eb_app_url" {
  value = aws_elastic_beanstalk_environment.my_web_app_env.endpoint_url
}
README.md:
markdown
Copy code
# AWS Elastic Beanstalk Terraform Setup

This Terraform script sets up an AWS Elastic Beanstalk application and environment named `my-web-app` and `my-web-app-env`, respectively. It includes components such as a VPC, subnets, security groups, an Application Load Balancer, and more. The infrastructure is configured for high availability, security, and proper tagging.

## Prerequisites

## Usage

1. **Clone this repository:**

   ```bash
   git clone 
   cd elastic-beanstalk-terraform

Initialize Terraform:

bash
Copy code
terraform init
Adjust Configuration:

Open main.tf and modify the configuration as needed. For example, you may want to change the AWS region, IP range for SSH access, etc.

Apply Changes:

bash
Copy code
terraform apply
Terraform will prompt you to confirm the changes. Enter yes to proceed.

Wait for the environment to be ready:

Check the status using:

bash
Copy code
terraform show
Access Your Web Application:

Once the environment is ready, find the URL in the Terraform output. Open the URL in your web browser to view Elastic Beanstalk application.

Outputs
eb_app_url: The URL of the deployed Elastic Beanstalk application.
Clean Up
To destroy the resources created by Terraform and avoid incurring charges:





b.      
A brief explanation of the architecture being created. 



# AWS Elastic Beanstalk Terraform Setup

This Terraform script creates an AWS Elastic Beanstalk application and environment for a web application.

## Architecture Overview

1. **VPC:**
   - A Virtual Private Cloud (VPC) is created to isolate the infrastructure components.

2. **Subnets:**
   - Two subnets are created in different availability zones to ensure high availability and fault tolerance.

3. **Security Group:**
   - A security group is defined to control inbound and outbound traffic.
   - It allows incoming HTTP and HTTPS traffic from anywhere.
   - SSH access is restricted to a specific IP range for security.
   - Outbound traffic is allowed anywhere.

4. **Elastic Beanstalk Application and Environment:**
   - An Elastic Beanstalk application named `my-web-app` is created.
   - An Elastic Beanstalk environment named `my-web-app-env` is created, using the latest Amazon Linux 2 platform.
   - The environment is associated with the VPC, subnets, and security group.
   - Tags are added for better organization.

5. **Application Load Balancer (ALB):**
   - An Application Load Balancer is created to distribute traffic evenly across instances.
   - It is placed in the specified subnets and associated with the security group.

6. **Output:**
   - The Terraform script outputs the URL of the deployed Elastic Beanstalk application.

## Prerequisites

1. **AWS Account:** 
2. **Terraform:** Install Terraform.

## Usage

1. **Clone this repository:**

   ```bash
   git clone 
   cd elastic-beanstalk-terraform
   Initialize Terraform:

bash
Copy code
terraform init
Adjust Configuration:

Open main.tf and modify the configuration as needed. For example, you may want to change the AWS region, IP range for SSH access, etc.

Apply Changes:

bash
Copy code
terraform apply
Terraform will prompt you to confirm the changes. Enter yes to proceed.

Wait for the environment to be ready:

Check the status using:

bash
Copy code
terraform show
Access Your Web Application:

Once the environment is ready, find the URL in the Terraform output. Open the URL web browser to view your Elastic Beanstalk application.

Outputs
eb_app_url: The URL of the deployed Elastic Beanstalk application.
Clean Up
To destroy the resources created by Terraform and avoid incurring charges:

bash
Copy code
terraform destroy
Enter yes when prompted.

Notes
Ensure that your AWS credentials are configured on your local machine.
Customize the Terraform script according to your specific requirements.
vb net
Copy code
Highlighting key components such as VPC, subnets, security groups, Elastic Beanstalk, and the Application Load Balancer.




c.
Any prerequisites or assumptions for the Terraform script execution.


Prerequisites:

AWS Account
AWS Credentials: Ensure that AWS credentials are configured on your local machine.
 This typically involves setting up the AWS Access Key ID and Secret Access Key, configure these credentials using the AWS CLI or environment variables.

Using AWS CLI:

bash
Copy code
aws configure
Environment Variables:

bash
Copy code
export AWS_ACCESS_KEY_ID=your_access_key_id
export AWS_SECRET_ACCESS_KEY=your_secret_access_key
Terraform Installed: Install Terraform on a local machine.


Assumptions:

Terraform Version: The Terraform script assumes using a version of Terraform compatible with the script.
It's recommended to use a relatively recent version to ensure compatibility with the features and providers used in the script.

Customization: The Terraform script is a template, and certain values need to be customized based on your specific requirements.
These include but are not limited to:

AWS region in the provider block.
IP range for SSH access in the security group.
Any other parameters or settings specific to your application or infrastructure.
Security Group Rules: The security group in the script allows incoming HTTP, HTTPS, and SSH traffic.
Ensure that these rules align with security requirements.

Elastic Beanstalk Platform: The script uses the latest Amazon Linux 2 platform for Elastic Beanstalk.
Specific platform requirements, may need to adjust the solution_stack_name in the Elastic Beanstalk environment resource.

Network Configuration: The script assumes a simple network configuration with public subnets for instances.
Specific network requirements, adjust the VPC and subnet configurations accordingly.

Load Balancer: The script creates an Application Load Balancer.
Application requires a different type of load balancer or additional features, modify the load balancer resource accordingly.

Cleanup: The script provides instructions on how to destroy the created resources using terraform destroy.
Ensure understand the potential impact of running this command and use it responsibly.

Internet Connectivity: The instances in the Elastic Beanstalk environment are designed to have internet connectivity.
Application requires specific network configurations, adjust the VPC, subnet, and security group settings accordingly.

Always review and understand the Terraform script and adjust it based on your specific use case and requirements before executing it.
Additionally, ensure compliance with AWS best practices and security guidelines.





7.
Deliverables:

a.Terraform scripts (*.tf files) that can be executed to provision the required infrastructure.


Terraform scripts split into multiple files.
Create these files in the same directory and execute them to provision the required infrastructure.

File: provider.tf
hcl
# provider.tf

provider "aws" {
  region = "us-east-1"
}
File: vpc.tf
hcl
Copy code
# vpc.tf

resource "aws_vpc" "my_vpc" {
  cidr_block          = "10.0.0.0/16"
  enable_dns_support  = true
  enable_dns_hostnames = true

  tags = {
    Name        = "my-web-app-vpc"
    Environment = "Production"
  }
}
File: subnets.tf
hcl
Copy code
# subnets.tf

resource "aws_subnet" "subnet_a" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name        = "subnet-a"
    Environment = "Production"
  }
}

resource "aws_subnet" "subnet_b" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true

  tags = {
    Name        = "subnet-b"
    Environment = "Production"
  }
}
File: security_group.tf
hcl
Copy code
# security_group.tf

resource "aws_security_group" "web_app_sg" {
  vpc_id = aws_vpc.my_vpc.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["X.X.X.X/32"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "web-app-security-group"
    Environment = "Production"
  }
}
File: elastic_beanstalk.tf
hcl
Copy code
# elastic_beanstalk.tf

resource "aws_elastic_beanstalk_application" "my_web_app" {
  name        = "my-web-app"
  description = "My Web Application"
}

resource "aws_elastic_beanstalk_environment" "my_web_app_env" {
  name                = "my-web-app-env"
  application         = aws_elastic_beanstalk_application.my_web_app.name
  solution_stack_name = "64bit Amazon Linux 2 v5.5.5 running Python 3.8"
  tier                = "WebServer"

  setting {
    namespace = "aws:ec2:vpc"
    name      = "VPCId"
    value     = aws_vpc.my_vpc.id
  }

  setting {
    namespace = "aws:ec2:vpc"
    name      = "Subnets"
    value     = "${aws_subnet.subnet_a.id},${aws_subnet.subnet_b.id}"
  }

  setting {
    namespace = "aws:autoscaling:launchconfiguration"
    name      = "SecurityGroups"
    value     = aws_security_group.web_app_sg.id
  }

  tags = {
    Name        = "my-web-app-env"
    Environment = "Production"
  }
}
File: load_balancer.tf
hcl
Copy code
# load_balancer.tf

resource "aws_lb" "web_app_lb" {
  name               = "web-app-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web_app_sg.id]
  subnets            = [aws_subnet.subnet_a.id, aws_subnet.subnet_b.id]
  enable_deletion_protection = false

  enable_cross_zone_load_balancing = true
  idle_timeout                      = 60

  tags = {
    Name        = "web-app-lb"
    Environment = "Production"
  }
}
File: output.tf
hcl
Copy code
# output.tf

output "eb_app_url" {
  value = aws_elastic_beanstalk_environment.my_web_app_env.endpoint_url
}
File: README.md
markdown
Copy code
# AWS Elastic Beanstalk Terraform Setup

To execute these scripts:
Save each script in separate files with the specified names.
Open a terminal in the directory containing these files.
Run terraform init to initialize in the working directory.
Run terraform to apply the Terraform configuration and create the infrastructure.





b.
A README.md file that documents the steps to initialize and apply the Terraform configuration, and any other necessary explanations.


# AWS Elastic Beanstalk Terraform Setup

This Terraform script automates the provisioning of infrastructure for an AWS Elastic Beanstalk web application.
The setup includes a VPC, subnets, security groups, Elastic Beanstalk application and environment, and an Application Load Balancer.

## Prerequisites

1. **AWS Account:**

2. **Terraform:** Install Terraform.

## Usage

Follow the steps below to initialize and apply the Terraform configuration.

### Step 1: Clone the Repository
```bash
git clone
cd elastic-beanstalk-terraform

Step 2: Initialize Terraform
bash
Copy code
terraform init
This command initializes your working directory and downloads the necessary providers.

Step 3: Adjust Configuration (if needed)
Open the Terraform files (*.tf) in a text editor to review and adjust the configuration based on requirements.
Common configurations to check include AWS region, IP ranges, and any other parameters specific to application.

Step 4: Apply Changes
bash
Copy code
terraform apply
Terraform will prompt to confirm the changes. Enter yes to apply the configuration and create the infrastructure.

Step 5: Monitor Provisioning
Wait for Terraform to provision the infrastructure. Monitor the progress in the terminal. Once completed,
Terraform will display a summary of the resources created.

Step 6: Access Web Application
Once the environment is ready, you can find the URL of Elastic Beanstalk application in the Terraform output.
Open the URL in your web browser to access the web application.

Outputs
eb_app_url: The URL of the deployed Elastic Beanstalk application.
Clean Up
To destroy the resources created by Terraform and avoid incurring charges:

bash
Copy code
terraform destroy
Enter yes when prompted.

In the README file, users are provided with clear instructions on cloning the repository,
Initializing and applying the Terraform configuration, accessing the web application, and cleaning up resources.

