---
title: "Build Golden Machine Images"
description: "Building AWS AMIs using Packer"
date: 2024-01-2T23:17:40-07:00
categories: ["aws"]
tags: ["how to"]
# weight: 1
draft: true
---

Golden Images bring consistency to the deployment of virtual machines. By capturing a pre-configured and fully tested snapshot of an operating system and application stack, Golden Images ensure that every instance created from them starts with the same baseline configuration. 

Golden Images allows us to use identical configurations between various environments, such as testing and production, ensuring that applications are tested in an environment that closely mirrors production.

Another benefit of using Golden Images when deploying VMs is simplifying troubleshooting and enhancing security by deploying instances with known, documented, `git` versioned, and validated configurations.

Without further ado, let's dive in and create our own simple AWS AMI image using Packer and Ansible.

## Prerequisites

- AWS Account (duh!)
- Hashicorp Packer
- AWS CLI - Install and configure the AWS CLI with the necessary credentials. You can install the AWS CLI by following the instructions in the official documentation.

## Step 1: Create Packer configuration file

You can use JSON or HCL for the config file. I will use HCL since it's the standardized way to do things in the Hashicrop ecosystem nowadays.

Content of `packer.hcl`

```hcl
variable "aws_access_key" {
  type    = string
  default = var.AWS_ACCESS_KEY_ID
}

variable "aws_secret_key" {
  type    = string
  default = var.AWS_SECRET_ACCESS_KEY
}

source "amazon-ebs" "myebsvolume" {
  access_key    = var.aws_access_key
  secret_key    = var.aws_secret_key
  region        = "your-aws-region"

  source_ami_filter {
    filters = {
      virtualization-type = "hvm"
      name                = "amzn2-ami-hvm-*-x86_64-gp2"
    }
    owners = ["amazon"]
    most_recent = true
  }

  instance_type = "t2.micro"
  ssh_username  = "ec2-user"
  
  # It's a good idea to use a timestamp in the AMI name
  ami_name      = "packer-myebsvolume-${timestamp()}"
  
  ami_description = "AMI created with Packer"
}

build {
  sources = ["source.amazon-ebs.myebsvolume"]

  provisioner "ansible" {
    playbook_file = "path/to/your/playbook.yml"
    # Add extra variables if needed
    extra_arguments = ["--extra-vars", "var_name=value"]  
  }
}
```

This configuration uses the official Amazon Linux 2 AMI as a base image. You can utilize provisioners other than Ansible by referring to the supported [provisioners documentation](https://developer.hashicorp.com/packer/docs/provisioners).

## Step 2: Build you Ansible playbooks
<!--Add a simple playbook-->

## Step 3: Build the Golden Image

Open a terminal and navigate to the directory containing your Packer configuration file and run the build command:

```sh
packer build packer.hcl
```

When the Packer job finishes, the AMI creation process is complete, and the AMI will be available in your AWS account. You can find the newly created AMI in the AWS Management Console under the "AMIs" section.

## Step 4: Clean Up

After the image is successfully created, make sure to clean up the resources to avoid unnecessary charges. You can do this by terminating the EC2 instance created during the build process.

<!--## Finally
Now you can use this AMI as a reliable when you provision your EC2 instances. 

You can explore additional Packer options within the [HashiCorp Packer documentation](https://developer.hashicorp.com/packer/docs) for further customization and expansion of the configuration to match your exact needs.
-->
