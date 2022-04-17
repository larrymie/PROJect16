# PROJect16
AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

After buiding AWS infrastructure for 2 websites manually, it is time to automate the process using Terraform. I will start building the same set up with the power of Infrastructure as Code (IaC) IMAGE 01
![01](https://user-images.githubusercontent.com/91284177/163706091-bdb567fb-8d66-4fe7-9416-0af500068f57.png)

**VPC | SUBNETS | SECURITY GROUPS

I created a directory structure. Also created a folder called PBL and created a file in the folder, named it main.tf IMAGE 02

![02](https://user-images.githubusercontent.com/91284177/163706395-9094800c-367e-4a99-bed1-2dabf4c6266a.png)

**Provider and VPC resource section

Add AWS as a provider, and a resource to create a VPC in the main.tf file.
Provider block informs Terraform that I intend to build infrastructure within AWS.
Resource block created a VPC in AWS Network as shown in the architectural design.

I used 'terraform fmt' to (arrange the code) used 'terraform plan' to (view the analysis of the structure i was about to provision) and 'terraform apply'
(for the eventual provision of the infrastructure) IMAGE 03 

![03](https://user-images.githubusercontent.com/91284177/163706602-aaa245bd-efb4-4fc4-82ee-3e3191d4e93d.png)

**Subnets resource section
According to the architectural design, required 6 subnets:

2 public
2 private for webservers
2 private for data layer
I created the first 2 public subnets in this project

**FIXING THE PROBLEMS BY CODE REFACTORING
Fixing The Problems By Code Refactoring
Fixing Hard Coded Values: I will introduce variables, and remove hard coding

Started with the provider block, declared a variable named region, gave it a default value, and updated the provider section by referring to the declared variable.

I used Data Sources to fetch information outside of Terraform. In this case, from AWS, fetched Availability zones from AWS, and replacde the hard coded value in the subnet’s availability_zone section.

 I introduced a count argument in the subnet block.
 
 I made cidr_block dynamic by introducing a function cidrsubnet() to make this happen. It accepts 3 parameters. I updated the configuration, then explore its internals.

I later removed hard coded count value.

I introuduced length() function, which basically determines the length of a given list, map, or string.
I updated the public subnet block.

What i have now, is sufficient to create the subnet resource required. But if you observe, it is not satisfying my business requirement of just 2 subnets. The length function will return number 3 to the count argument, but what we actually need is 2.

I fixed this by declaring a variable to store the desired number of public subnets, and set the default value.

finally i updated the count argument with a condition. Terraform needs to check first if there is a desired number of subnets. Otherwise, use the data returned by the lenght function.

My structure looked like this after the whole editing:

# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

variable "region" {
      default = "eu-west-2"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = 2
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}

# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}


**INTRODUCING VARIABLES.TF & AMP; TERRAFORM.TFVARS
Introducing variables.tf & terraform.tfvars

Instead of havng a long lisf of variables in main.tf file, I made my code a lot more readable and better structured by moving out some parts of the configuration content to other files.

I put all variable declarations in a separate file and provided non default values to each of them. I also created a new file and named it **variables.tf
Copied all the variable declarations into the new file.Created another file, named it terraform.tfvars set values for each of the variables. IMAGES 04, 05 & 06

![04](https://user-images.githubusercontent.com/91284177/163707945-e8466cb2-45a8-4883-a0ce-212a397eeb52.png)

 ![05](https://user-images.githubusercontent.com/91284177/163707948-4c8634ae-0642-4a62-938b-56c6c36b791f.png)

![06](https://user-images.githubusercontent.com/91284177/163707950-4f44ac17-44c6-445a-9a45-cd66f04d1b91.png)

I logged in to my AWS platform console to confirm if my provisioned infrastrctures (VPC, 2 public subnets) were executed as applied via terraform. it was successfully done. IMAGES 07 & 08.
![07](https://user-images.githubusercontent.com/91284177/163708183-3c9d818f-b624-4d8d-b8aa-157cbf1cbb2f.png)

![08](https://user-images.githubusercontent.com/91284177/163708213-b7f624c9-6b52-4243-9c12-b21d0ed3c548.png)


















