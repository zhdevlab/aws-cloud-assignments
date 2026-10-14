# Assignment 1: VPC & Networking

Built a custom network from scratch on AWS: a VPC with public and private subnets, correct routing, and EC2 instances secured so the private one is only reachable through a bastion host.

## Architecture

![Architecture diagram](./architecture/architecture-diagram.png)

## What I built

- Custom VPC (`aws-vpc`)
- Two subnets: one public, one private
- Internet Gateway and NAT Gateway (with Elastic IP)
- Two route tables: public (via IGW) and private (via NAT Gateway)
- Two EC2 instances: one public, one private
- Two security groups: public and private
- Bastion host to access the private EC2
- CloudWatch monitoring on the public EC2

## Step by step

### 1. Create the VPC
Created the VPC and named it `aws-vpc`.

![VPC created](./screenshots/01-vpc-created.png)

### 2. Create the subnets
Created two subnets: `aws-vpc-public-subnet-01` and `aws-vpc-private-subnet-01`.

![Subnets created](./screenshots/02-subnets-created.png)

### 3. Internet Gateway
Created the Internet Gateway (`aws-vpc-igw`) and attached it to the VPC.

![Internet Gateway attached](./screenshots/03-igw-attached.png)

### 4. Elastic IP and NAT Gateway
Allocated an Elastic IP, then created the NAT Gateway and placed it in the public subnet.

![NAT Gateway created](./screenshots/04-nat-gateway.png)

### 5. Route tables
Created two route tables, one public and one private, and associated each with its matching subnet.

- Public route table: default route (`0.0.0.0/0`) to the Internet Gateway
- Private route table: default route (`0.0.0.0/0`) to the NAT Gateway

![Public route table](./screenshots/05-route-table-public.png)
![Private route table](./screenshots/06-route-table-private.png)

### 6. Public security group
Created `aws-vpc-ssh-http-sg` in `aws-vpc`. Inbound SSH and HTTP are allowed only from my public IP. Outbound is set to all traffic.

![Public security group](./screenshots/07-sg-public.png)

### 7. Public EC2 instance
Launched the public EC2 in the public subnet with auto-assign public IP enabled and the security group above attached.

![Public EC2 running](./screenshots/08-ec2-public.png)

### 8. Private security group
Created a second security group for the private EC2. SSH is only allowed from the public EC2's security group (used as the source), not from an IP address.

![Private security group](./screenshots/09-sg-private.png)

### 9. Private EC2 instance
Launched the private EC2 in the private subnet with no public IP.

![Private EC2 running](./screenshots/10-ec2-private.png)

### 10. Bastion host
Created a bastion instance in the public subnet so the private EC2 can be reached through a single, controlled entry point.

![Bastion host running](./screenshots/11-bastion-host.png)

### 11. CloudWatch monitoring
Enabled CloudWatch monitoring on the public EC2.Currently have not set any notification as this is a demo. 

![CloudWatch metrics](./screenshots/12-cloudwatch.png)

## Testing

| Test | Result |
|------|--------|
| SSH from my machine (Ubuntu) to the public EC2 | Successful |
| HTTP to the public EC2 | Timed out, as no web server was installed yet |
| SSH from my machine straight to the private EC2 | Failed, as expected |
| SSH to the public EC2, then on to the private EC2 | Successful |
| SSH to the bastion, then on to the private EC2 | Successful |

![SSH to public EC2](./screenshots/13-test-ssh-public.png)
![Direct SSH to private EC2 fails](./screenshots/14-test-ssh-private-blocked.png)
![SSH via bastion to private EC2](./screenshots/15-test-ssh-via-bastion.png)

## Problems I hit and how I fixed them

**Problem:** I couldn't SSH into the private EC2 from the public EC2.

**Cause:** I had attached it to the first security group, which only allowed SSH from my own IP.

**Fix:** Created a separate security group for the private EC2, using the public EC2's security group as the source, and attached that instead.

## Clean-up and cost

To avoid ongoing charges until the next assignment, I stopped the instances, released the Elastic IP and deleted the NAT Gateway.
