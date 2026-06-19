Terraform AWS ALB + ASG with Path-Based Routing
A Terraform project that provisions an Application Load Balancer (ALB) with path-based routing across two Auto Scaling Groups — one for the Home page and one for the Mobile page.

Architecture
Internet
    │
    ▼
Application Load Balancer (lb-cbz)
    │
    ├── / (default)       ──▶  Target Group: tg-home   ──▶  ASG: asg-home   ──▶  EC2 (Home Page)
    │
    └── /mobile/*         ──▶  Target Group: tg-mobile ──▶  ASG: asg-mobile ──▶  EC2 (Mobile Page)
Resources Created
Resource	Name	Description
Launch Template	lt-home	Ubuntu + Apache serving Home Page
Launch Template	lt-mobile	Ubuntu + Apache serving Mobile Page
Target Group	tg-home	Health check on /
Target Group	tg-mobile	Health check on /mobile
Application Load Balancer	lb-cbz	Public-facing ALB
ALB Listener	cbz-listener	Listens on port 80
Listener Rule	cbz-listener-rule	Routes /mobile/* to tg-mobile
Auto Scaling Group	asg-home	Min:1, Max:3, Desired:1
Auto Scaling Group	asg-mobile	Min:1, Max:3, Desired:1
Scaling Policy	policy-home	Target tracking — 70% CPU
Scaling Policy	policy-mobile	Target tracking — 70% CPU
Path-Based Routing Logic
URL Path	Routes To
http://<alb-dns>/	Home Page (tg-home)
http://<alb-dns>/mobile	Mobile Page (tg-mobile)
http://<alb-dns>/mobile/*	Mobile Page (tg-mobile)
Auto Scaling Policy
Both ASGs use Target Tracking Scaling based on average CPU utilization:

CPU > 70% → AWS automatically adds instances (scale out)
CPU < 70% → AWS automatically removes instances (scale in)
No manual CloudWatch alarms needed — AWS manages it internally
Prerequisites
Terraform installed
AWS CLI configured (aws configure)
An existing:
VPC
Two public Subnets (in different AZs)
Security Group (allow port 80 inbound)
Key Pair
File Structure
project/
├── main.tf           # All AWS resources
├── variables.tf      # Variable declarations
└── .gitignore        # Excludes .terraform/, tfstate, tfvars
Variables
Variable	Description	Example
image	AMI ID for EC2 instances	ami-0b6d9d3d33ba97d99
type	EC2 instance type	t3.micro
key	Key pair name for SSH	my-key
sg	Security Group ID	sg-xxxxxxxxxxxxxxxxx
vpc	VPC ID	vpc-xxxxxxxxxxxxxxxxx
Usage
bash
# Initialize Terraform
terraform init

# Preview changes
terraform plan

# Apply infrastructure
terraform apply

# Destroy infrastructure
terraform destroy
Access the Application
After terraform apply, get the ALB DNS name:

bash
terraform state show aws_lb.lb_cbz | grep dns_name
Then open in browser:

http://<alb-dns-name>/         → Home Page
http://<alb-dns-name>/mobile   → Mobile Page
.gitignore
.terraform/
.terraform.lock.hcl
terraform.tfstate
terraform.tfstate.backup
*.tfvars
⚠️ Never push .terraform/ to GitHub — it contains provider binaries (800MB+). Only .tf files should be committed.

Key Concepts Learned
Launch Templates — Define EC2 config (AMI, instance type, user data) for ASG use
Target Groups — Group of instances that receive traffic from ALB
Path-Based Routing — Route different URL paths to different backend services
Auto Scaling Groups — Automatically manage EC2 instance count based on demand
Target Tracking Policy — Simplest scaling policy; AWS auto-scales to maintain target metric

