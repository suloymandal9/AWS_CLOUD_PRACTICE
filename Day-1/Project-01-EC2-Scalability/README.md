# AWS EC2 Custom AMI Creation and Instance Replication

## Project Overview

This project demonstrates how to configure an AWS EC2 Linux instance with Apache HTTP Server, create a custom Amazon Machine Image (AMI), and launch a second EC2 instance using the custom AMI.

The purpose is to replicate a preconfigured web server without manually installing and configuring the software again.

## Technologies Used

- AWS EC2
- Amazon Linux 2023
- Apache HTTP Server
- MobaXterm
- SSH
- Linux Commands
- AWS Security Groups
- Custom Amazon Machine Image (AMI)
- HTML

## Implementation Steps

### 1. Launch EC2 Instance

- Launched an Amazon Linux 2023 EC2 instance.
- Selected an appropriate instance type.
- Created or selected an SSH key pair.
- Configured SSH access through Security Groups.

### 2. Connect Using MobaXterm

Connected to the Linux EC2 instance using SSH and the PEM private key.

### 3. Install Apache

```bash
sudo dnf update -y
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```

### 4. Create a Custom Website

Created an HTML file:

```bash
sudo vi /var/www/html/index.html
```

Added website content and saved the file.

Verified the website locally:

```bash
curl http://localhost
```

### 5. Create Custom AMI

- Selected the original EC2 instance.
- Navigated to Actions → Image and templates → Create image.
- Created a custom AMI named `My-Custom-Linux-AMI`.
- Waited for the AMI status to become Available.

### 6. Launch a New Instance from Custom AMI

- Selected the newly created custom AMI.
- Launched a second EC2 instance.
- Selected the existing SSH key pair.
- Configured the required Security Group rules.

### 7. Verify the New Instance

Connected to the second instance using MobaXterm.

Executed:

```bash
sudo systemctl status httpd
ls -l /var/www/html/
curl http://localhost
```

Verified that Apache and the website files were already available from the custom AMI.

### 8. Browser Verification

Accessed the website through:

```text
http://NEW-EC2-PUBLIC-IP
```

Allowed HTTP traffic on TCP port 80 through the Security Group.

## Troubleshooting and Issues Faced

### Issue: Website Not Reachable

**Error:** Chrome displayed "This site can't be reached."

**Debugging steps:**

1. Checked Apache service using `systemctl status httpd`.
2. Verified website response using `curl http://localhost`.
3. Confirmed Apache was running and the HTML content was available.
4. Inspected EC2 Security Group inbound rules.
5. Identified that HTTP port 80 was missing.

**Solution:**

Added an HTTP inbound rule:

- Type: HTTP
- Protocol: TCP
- Port: 80
- Source: 0.0.0.0/0

This allowed browser requests to reach Apache.

## Key Learnings

- Understanding Amazon Machine Images (AMI).
- Creating custom AMIs from configured EC2 instances.
- Launching multiple EC2 instances from a custom AMI.
- Configuring Apache HTTP Server on Amazon Linux.
- Managing EC2 instances using MobaXterm and SSH.
- Understanding SSH (22) and HTTP (80) ports.
- Configuring AWS Security Groups.
- Troubleshooting website connectivity issues.

## Project Outcome

Successfully prepared an Apache web server on AWS EC2, created a custom AMI, and used it to launch a second instance with the existing web server configuration and website files.

## Author

AWS Cloud and DevOps Hands-on Practice
