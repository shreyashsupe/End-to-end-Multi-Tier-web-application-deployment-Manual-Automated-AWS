# End-to-end Multi Tier web application deployment (Manual -> Automated -> AWS)

This repository is a learning project that demonstrates deploying a multi-tier Java web application through three stages:
1. Manual local deployment (classic setup with Tomcat, MySQL, Memcache, RabbitMQ)
2. Automated local deployment using Vagrant (multi-VM environment, service setup scripts)
3. AWS "Lift-and-Shift" architecture and guidance for migrating the same multi-tier app to AWS

![Project Architecture](https://github.com/shreyashsupe/End-to-end-Multi-Tier-web-application-deployment-Manual-Automated-AWS/blob/main/Architecture_Diagram.png)


## Quick links

- Manual setup webapp source: Manual Setup/ (see pages and Java code)
- Automated setup guide: [Automated Setup/Automated_setup_Guide.txt](https://github.com/shreyashsupe/End-to-end-Multi-Tier-web-application-deployment-Manual-Automated-AWS/blob/main/Automated%20Setup/Automated_setup_Guide.txt)
- AWS migration guide: [AWS-Lift-and-Shift/README.md](https://github.com/shreyashsupe/End-to-end-Multi-Tier-web-application-deployment-Manual-Automated-AWS/blob/main/AWS-Lift-and-Shift/README.md)

---

## Project overview and goals
- Show a typical multi-tier web application stack:
  - Java web application (Spring + JSP) running on Tomcat
  - Relational DB (MySQL/MariaDB)
  - Memcache (caching)
  - RabbitMQ (message broker)
- Provide:
  - Manual configuration/examples so you can understand each component and how they integrate
  - An automated Vagrant-based environment that brings up multiple VMs and configures services with shell scripts
  - Documentation and architecture notes for migrating the same stack to AWS (lift-and-shift approach)
- Learning outcomes:
  - Understand component roles and interactions
  - Learn how to automate provisioning, install services, and deploy a WAR
  - Understand the AWS services that map to each component (EC2, ALB, ASG, S3, IAM, ACM, Route53)
  - Practice debugging and troubleshooting multi-tier integration problems

---

## How to run the project — Manual Setup 
This is useful to learn how components are configured by hand.
- cd into Manual Setup
- Refer the Manual Setup Guide to do the complete setup of the servers one by one [Manual_setup_Guide](https://github.com/shreyashsupe/End-to-end-Multi-Tier-web-application-deployment-Manual-Automated-AWS/blob/main/Manual%20Setup/ManualProject_Setup_Guide.txt)
- Once the setup is done validate the working of the web app

---

## How to run the project — Automated Setup 
This is the recommended route for reproducing a production-like multi-VM environment similar to a small data center.

Steps:
1. Open a terminal and go to the `Automated Setup` folder:
   - cd "Automated Setup"
   - View the guide: `Automated_setup_Guide.txt`  
     - https://github.com/shreyashsupe/End-to-end-Multi-Tier-web-application-deployment-Manual-Automated-AWS/blob/main/Automated%20Setup/Automated_setup_Guide.txt
2. Bring up the environment:
   - vagrant up
   - This will create multiple VMs (examples: db01, memcache, rabbitmq, app servers) and run the provisioning scripts.
3. The provisioning scripts:
   - `mysql.sh` — installs & configures MariaDB
   - `memcache.sh` — installs memcached
   - `rabbitmq.sh` — installs RabbitMQ
   - `tomcat.sh` / `tomcat_ubuntu.sh` — installs Tomcat, deploys the WAR
   - `nginx.sh` — optional reverse proxy configuration
4. After provisioning finishes, access the app using the IP / hostname configured by Vagrant (see Automated guide for IP/hosts)

What the automation does (summary from guide):
- Creates VMs, updates /etc/hosts, installs & configures all services, builds and deploys the application automatically.

Note: This section includes both hands-on implementation steps and architectural guidance for lift-and-shift migration to AWS.

---

## AWS Lift-and-Shift
Purpose: show how to migrate the same multi-tier app with minimal code changes into AWS.

Key ideas (documented in `AWS-Lift-and-Shift/README.md`):
- Keep the application as-is and run it on EC2 instances (no refactor) — "lift-and-shift"
- Replace local infrastructure with AWS equivalents:
  - EC2 instances for Tomcat and backend services
  - ALB (Application Load Balancer) instead of Nginx
  - Auto Scaling Group (ASG) for scaling app servers
  - RDS (or EC2-hosted MariaDB) for MySQL (RDS recommended)
  - ElastiCache for Memcached
  - Amazon MQ or managed RabbitMQ if needed (or run on EC2)
  - S3 for artifacts and static assets
  - Route 53 for DNS; IAM and ACM for credentials and certificates
- The AWS folder contains diagrams and notes to help design the migrated architecture.

---

## Common development and troubleshooting tips
- Check logs:
  - Tomcat logs in `logs/` on Tomcat host
  - MySQL logs if DB connection errors occur
  - Application logs printed by Spring to console or configured log files
- DB connectivity issues:
  - Confirm `application.properties` values and that DB user has privileges and DB exists
- Port conflicts:
  - Ensure ports used by Tomcat, MySQL, RabbitMQ, memcached are available
- Vagrant provisioning fails:
  - Rerun `vagrant reload --provision` after fixing the provisioning script or Vagrantfile
- Increasing VM RAM/disk:
  - Edit Vagrantfile or VirtualBox settings if provisioning fails due to resource constraints

Read the lift-and-shift doc for details:
- https://github.com/shreyashsupe/End-to-end-Multi-Tier-web-application-deployment-Manual-Automated-AWS/blob/main/AWS-Lift-and-Shift/README.md

---

## Contact & Links

- **LinkedIn**: www.linkedin.com/in/shreyashsupe
- **Email**: shreyashsupe11@gmail.com 
