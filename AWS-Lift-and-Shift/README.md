## AWS Lift and Shift – Multi-Tier Web Application Deployment

# Overview

This project demonstrates a Lift and Shift (Rehost) cloud migration strategy by moving an existing multi-tier Java web application from a virtualized/on-premise setup to the AWS Cloud without redesigning the application.

The objective is to run the same application architecture in a production-ready AWS environment, leveraging AWS-managed services for scalability, availability, and security.

This is Part 3 of the overall project journey:

Manual Setup → Automated Setup → AWS Lift & Shift

## What is Lift and Shift?

Lift and Shift is a cloud migration approach where:

- The application is moved as-is to the cloud
- No major code or architectural changes are made
- Existing configurations and dependencies are preserved

This strategy is ideal when:
- Quick migration is required
- Refactoring is not immediately feasible
- The application is already stable

## Architecture Overview
### Application Components

The application follows a multi-tier architecture:
- Tomcat – Java Application Server
- MySQL (MariaDB) – Relational Database
- Memcache – Caching Layer
- RabbitMQ – Message Broker

### AWS Services Used

- EC2 – Hosts application and backend services
- Application Load Balancer (ALB) – Handles incoming traffic (replaces Nginx)
- Auto Scaling Group (ASG) – Scales application instances automatically
- Route 53 – DNS-based service discovery
- S3 – Artifact storage
- IAM – Access control and permissions
- ACM – SSL/TLS certificates for HTTPS

## Architecture Diagram 
![Project Architecture](https://github.com/shreyashsupe/End-to-end-Multi-Tier-web-application-deployment-Manual-Automated-AWS/blob/main/AWS-Lift-and-Shift/Lift_%26_Shift_Architecture.png)

## Security Groups and Key Pairs
### Security Groups

We create three security groups:

1️⃣ Load Balancer Security Group  
**Inbound:**  
- HTTP (80) – from anywhere  
- HTTPS (443) – from anywhere  
  
**Outbound:**  
- Allow all traffic  

2️⃣ Application (Tomcat) Security Group  
**Inbound:**  
- Port 8080 – allowed only from ALB security group  
- SSH (22) – allowed from your public IP  
  
**Outbound:**  
- Allow all traffic  

3️⃣ Backend Services Security Group  
**Inbound:**  
- MySQL (3306) – from Tomcat SG  
- Memcache (11211) – from Tomcat SG  
- RabbitMQ (5672) – from Tomcat SG  
- SSH (22) – from your public IP  
  
**Outbound:**  
- Allow all traffic

### Key Pair

Create an EC2 key pair  
Download the .pem file  
Used for SSH access to EC2 instances

## EC2 Instances Setup
### Instances Launched
| Service       | Purpose         | Security Group |  
| --------------|-----------------|----------------|
| MySQL        | Database        | Backend SG     |
| Memcache     | Cache           | Backend SG     |
| RabbitMQ     | Message Broker   | Backend SG     |
| Tomcat       | Application Server|Application SG  |

### Provisioning

All EC2 instances are provisioned using User Data scripts  
Services are installed and started automatically during launch

## DNS Configuration with Route 53
### Why Route 53?

Replaces /etc/hosts used in Vagrant  
Enables DNS-based service communication  
Production-grade name resolution

### Dummy Domain Example

We use a dummy domain for demonstration:
- example.internal  

### DNS Records Created
| Service       | DNS Record                    |
|----------------|------------------------------|
| MySQL          | db01.example.internal        |
| Memcache       | mc01.example.internal        |
| RabbitMQ       | rmq01.example.internal       |
| Tomcat         | app01.example.internal       |

These DNS names are referenced in application.properties.

### Verification

SSH into any instance and test:
- ping db01.example.internal  
- ping mc01.example.internal

## Build and Deploy Application Artifacts
### Build Artifact (Local Machine)

**Prerequisites:**  
- JDK  
- Maven  
- AWS CLI configured  

```bash
mvn install
```

This generates:
target/vprofile-v2.war

####  Upload Artifact to S3
```bash
aws s3 cp target/vprofile-v2.war s3://example-ls-artifacts
```

####  Deploy Artifact on Tomcat EC2
- Attach an IAM Role with S3 access to the Tomcat instance
- Install AWS CLI on the Tomcat server

```bash
aws s3 cp s3://example-ls-artifacts/vprofile-v2.war /tmp/ 
```

#### Deploy the application:

```bash
systemctl stop tomcat10
rm -rf /var/lib/tomcat10/webapps/ROOT
cp /tmp/vprofile-v2.war /var/lib/tomcat10/webapps/ROOT.war
systemctl start tomcat10 
```
Tomcat automatically extracts and runs the application.

### Load Balancer and HTTPS Setup

#### Application Load Balancer
- Target Group:
    - Port 8080
    - Health checks enabled
- HTTPS
    - SSL certificate created using AWS Certificate Manager
    - Domain validation via Route 53
    - HTTPS enabled on ALB listener (443)

### Auto Scaling Group
To ensure availability and scalability:

##### Steps:
1. Create an AMI from the configured Tomcat instance
2. Create a Launch Template from the AMI
3. Create an Auto Scaling Group
    - Minimum: 1
    - Desired: 2
    - Maximum: As per load
4. Attach ASG to ALB target group

The application now:
    - Scales out on high traffic
    - Scales in during low traffic