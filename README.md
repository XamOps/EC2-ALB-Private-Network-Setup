# AWS Cloud Infrastructure Deployment Script for Web Applications

This Bash script automates the provisioning of essential AWS infrastructure components to deploy a web application securely on an EC2 instance within a private subnet, accessible via an Application Load Balancer (ALB).

## Table of Contents
- [Project Overview](#project-overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
- [Usage](#usage)
- [Script Output](#script-output)
- [Next Steps After Infrastructure Deployment](#next-steps-after-infrastructure-deployment)
- [Important Notes](#important-notes)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Project Overview

This script is designed to rapidly provision a standard, secure, and scalable AWS infrastructure pattern for web applications. It sets up the networking, security, and compute layers, ensuring your application's backend instances remain private while being accessible to end-users through a load balancer.

## Features

* **Dynamic Resource Naming:** All resources are named using a configurable prefix for easy identification.
* **Multi-AZ Subnets:** Provisions public and private subnets across two Availability Zones for high availability.
* **Internet & NAT Gateway Setup:** Configures internet access for public subnets and outbound internet access for private subnets via a NAT Gateway.
* **Custom Route Tables:** Sets up dedicated public and private route tables.
* **Robust Security Group Configuration:** Creates specialized security groups for the ALB (public access) and EC2 instances (ALB-only HTTP access, limited SSH access).
* **Dynamic AMI Selection:** Automatically selects the latest Ubuntu 24.04 LTS (Noble Numbat) AMI for the EC2 instance.
* **Application Load Balancer (ALB) Setup:** Deploys an ALB, Target Group, and HTTP Listener, forwarding traffic to your private EC2 instance.
* **Basic EC2 User Data:** Includes a simple user data script to install Apache and serve a placeholder page on the EC2 instance upon launch.

## Prerequisites

Before running this script, ensure you have:

* **AWS CLI Configured:** The AWS Command Line Interface must be installed and configured with credentials that have sufficient permissions to create, modify, and describe EC2, VPC, and ELBv2 resources in your chosen AWS region.
* **Existing VPC ID:** You must have an existing VPC where you want to deploy this infrastructure. Its ID will be specified in the script's configuration.
* **Existing EC2 Key Pair:** An SSH Key Pair must exist in your AWS account in the target region. Its name will be specified in the script's configuration for SSH access to the EC2 instance.

## Configuration

Open the `deploy_aws_infra.sh` file in a text editor and customize the variables in the `--- Configuration Variables ---` section:

* `AWS_REGION`: The AWS region where resources will be deployed (e.g., `us-east-1`, `ap-south-1`).
* `VPC_ID_TO_USE`: **Your existing VPC ID** where all resources will be created (e.g., `vpc-xxxxxxxxxxxxxxxxx`).
* `INSTANCE_TYPE`: The desired EC2 instance type (e.g., `t3.small`, `t2.micro`).
* `KEY_PAIR_NAME`: The name of your existing EC2 Key Pair (e.g., `my-ssh-key`).
* `APP_PORT`: The application port on your EC2 instance (usually `80` for HTTP, `443` for HTTPS).
* `RESOURCE_NAME_PREFIX`: A **unique prefix** for all resources created by this script (e.g., `myproject-dev-app`). **Change this for each new deployment** to avoid naming conflicts.

## Usage

1.  **Save the script:** Save the provided Bash script content into a file (e.g., `deploy_aws_infra.sh`).
2.  **Make executable:**
    ```bash
    chmod +x deploy_aws_infra.sh
    ```
3.  **Run the script:**
    ```bash
    ./deploy_aws_infra.sh
    ```
    The script will output its progress and crucial information upon completion.

## Script Output

Upon successful execution, the script will print a summary including:

* The `RESOURCE_NAME_PREFIX` used.
* The `VPC_ID` where resources were deployed.
* IDs of created subnets (public and private).
* The EC2 Instance ID.
* The **ALB DNS Name**, which is the public URL for your application.

## Next Steps After Infrastructure Deployment

This script only sets up the AWS infrastructure. To deploy your actual application (like SuiteCRM) onto the EC2 instance:

1.  **Verify ALB Access:** Open the provided `http://<ALB_DNS_NAME>` in your web browser. You should see the "Hello from Private EC2 Instance!" placeholder page. This confirms your infrastructure is working.
2.  **SSH into EC2 Instance:** Use your configured `KEY_PAIR_NAME` and the EC2 Instance ID (from the script output) to SSH into the instance.
3.  **Run Application Deployment Script:** Execute your application-specific deployment script (e.g., `deploy_suitecrm.sh` from our previous discussion) on the EC2 instance to install and configure your software.

## Important Notes

* **Clean-up:** This script **does not** include a clean-up mechanism. To remove resources, you'll need to manually delete them via the AWS Console or AWS CLI in reverse order of creation (e.g., delete listener, then target group, then ALB, then EC2, then security groups, then route tables, then NAT Gateway, then subnets). Consider using AWS CloudFormation or Terraform for robust infrastructure lifecycle management.
* **SSH Access:** For production environments, tighten the SSH security group rule (port 22) to allow access only from specific IP addresses or a bastion host, not `0.0.0.0/0`.
* **HTTPS:** This script sets up an HTTP listener. For production, configure HTTPS on your ALB using AWS Certificate Manager (ACM).

## Troubleshooting

* **Script fails:** Check the output for specific AWS CLI error messages. Ensure your AWS CLI credentials have sufficient permissions.
* **"VPC ID is invalid"**: Double-check the `VPC_ID_TO_USE` variable in the script.
* **"Could not find a suitable AMI"**: Verify `AWS_REGION` is correct and that Ubuntu 24.04 AMIs are available there.
* **ALB DNS not resolving / "Page not found"**:
    * Allow a few minutes for ALB health checks to pass after EC2 instance launch.
    * Verify the EC2 instance is `running` and has passed health checks in the AWS console.
    * Check your security group rules.

## License

This project is open-source and available under the [MIT License](LICENSE).
