# Task-027: Create Application Load Balancer (ALB) and Distribute Traffic Between EC2 Instances

## Objective

Create an AWS Application Load Balancer (ALB), register two EC2 instances as targets, and verify that incoming HTTP traffic is distributed between both web servers.

## AWS Services Used

- Amazon EC2
- Application Load Balancer (ALB)
- Target Groups
- Security Groups
- Apache HTTP Server

## Environment Details

| Configuration | Value |
|---|---|
| AWS Region | US East (N. Virginia) – `us-east-1` |
| Load Balancer Name | `ALB-Demo` |
| Load Balancer Type | Application Load Balancer |
| Scheme | Internet-facing |
| Listener | HTTP – Port 80 |
| Target Group | `ALB-TG` |
| Target Protocol | HTTP |
| Target Port | 80 |
| Health Check Path | `/` |
| Web Server 1 | `primary_new` |
| Web Server 2 | `ALB-Web-Server-2` |

## Step 1: Launch Two EC2 Instances

Created two Amazon Linux EC2 instances in different Availability Zones.

- Server 1: `primary_new` – `us-east-1a`
- Server 2: `ALB-Web-Server-2` – `us-east-1c`

Both instances were launched in the same VPC.

## Step 2: Install and Configure Apache HTTP Server

Installed and configured Apache HTTP Server on both EC2 instances.

**Server 1:**

```bash
sudo dnf update -y
sudo dnf install httpd -y
sudo systemctl enable --now httpd
```

Created a unique webpage:

```bash
echo "<h1>Welcome to ALB Web Server 1 - primary_new</h1>" | sudo tee /var/www/html/index.html
```

**Server 2:**

Configured Apache using EC2 User Data during instance launch.

```bash
#!/bin/bash
dnf update -y
dnf install httpd -y
systemctl enable httpd
systemctl start httpd

echo "<h1>Welcome to ALB Web Server 2</h1>" > /var/www/html/index.html
```

Each server displays a different message to identify which EC2 instance responds to a request.

## Step 3: Configure Security Groups

Created two separate security groups to control traffic between the internet, ALB, and EC2 instances.

### 1. ALB-SG

Attached this security group to the Application Load Balancer.

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | `0.0.0.0/0` |

**Purpose:** Allows users from the internet to access the internet-facing ALB through HTTP port 80.

### 2. ALB-Web-SG

Attached this security group to both EC2 web servers.

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | My IP |
| HTTP | TCP | 80 | `ALB-SG` |

**Purpose:** Allows HTTP traffic from the ALB security group to reach the EC2 web servers. SSH access is allowed from my IP for administration.

This configuration prevents direct public HTTP access to the backend web servers through this security group.

## Step 4: Create Target Group

Created a target group named `ALB-TG`.

Configuration:

- Target type: Instances
- Protocol: HTTP
- Port: 80
- VPC: Same VPC as the EC2 instances
- Health check protocol: HTTP
- Health check path: `/`

Registered both EC2 instances:

1. `primary_new`
2. `ALB-Web-Server-2`

After registration, verified that both targets showed **Healthy** status.

## Step 5: Create Application Load Balancer

Created an Application Load Balancer named `ALB-Demo`.

Configuration:

- Load balancer type: Application Load Balancer
- Scheme: Internet-facing
- IP address type: IPv4
- VPC: Same VPC as the EC2 instances
- Availability Zones: `us-east-1a` and `us-east-1c`
- Security group: `ALB-SG`

Configured the listener:

| Listener | Action |
|---|---|
| HTTP : 80 | Forward to `ALB-TG` |

Created the ALB and waited until its state became **Active**.

## Step 6: Verify Traffic Distribution

Copied the DNS name from the ALB details page.

Opened the following URL in a browser:

```text
http://<ALB-DNS-NAME>
```

Refreshed the browser multiple times and verified that both server responses appeared.

**Server 1 response:**

```text
Welcome to ALB Web Server 1 - primary_new
```

**Server 2 response:**

```text
Welcome to ALB Web Server 2
```

This confirmed that the ALB was successfully forwarding incoming HTTP requests to both registered EC2 instances.

Note: ALB does not guarantee that every consecutive browser refresh will alternate between the two servers.

## Architecture / Traffic Flow

```text
                 Internet / Browser
                         |
                         |
                    HTTP : 80
                         |
                         v
              Application Load Balancer
                    ALB-Demo
                    (ALB-SG)
                         |
                         |
                 Target Group
                   ALB-TG
                         |
               +---------+---------+
               |                   |
               v                   v
          EC2 Server 1        EC2 Server 2
          primary_new       ALB-Web-Server-2
               |                   |
               v                   v
          Apache HTTP          Apache HTTP
          Server : 80          Server : 80
               |                   |
               v                   v
          Server 1 Page       Server 2 Page
```

## Key Learnings

1. An Application Load Balancer works at Layer 7 (Application Layer).
2. ALB distributes incoming HTTP/HTTPS traffic across registered targets.
3. A target group contains the backend resources that receive traffic from the load balancer.
4. Health checks help ALB identify healthy and unhealthy targets.
5. Security groups control communication between the ALB and EC2 instances.
6. ALB can distribute traffic across EC2 instances running in different Availability Zones.
7. The ALB DNS name provides a common endpoint for accessing the application without directly accessing individual EC2 instances.

## Interview Questions and Answers

**Q1. What is an Application Load Balancer?**

An Application Load Balancer is a Layer 7 load balancer that distributes HTTP and HTTPS traffic across multiple targets based on configured listener rules.

**Q2. What is a Target Group?**

A target group is a collection of registered backend resources, such as EC2 instances, to which the load balancer forwards requests.

**Q3. What is a Health Check?**

A health check is a mechanism used by the load balancer to verify whether registered targets are available to receive traffic.

**Q4. Why do we use two security groups?**

We separate ALB and EC2 security groups to control traffic independently. The ALB security group allows public HTTP access, while the EC2 security group allows HTTP traffic only from the ALB security group.

**Q5. What happens if one EC2 instance becomes unhealthy?**

The ALB stops routing new requests to that unhealthy target and continues forwarding requests to healthy targets.

**Q6. Why do we use an ALB instead of accessing EC2 instances directly?**

An ALB provides a common endpoint, distributes incoming traffic, performs health checks, and improves application availability.

## Final Result

Successfully created an Application Load Balancer and registered two EC2 web servers in different Availability Zones.

Verified that both targets were healthy and that the ALB DNS name successfully displayed responses from both web servers.

**Status: Completed**

## Screenshots

Add your practical screenshots below:

1. Two EC2 instances running in different Availability Zones.
2. ALB-SG inbound rules.
3. ALB-Web-SG inbound rules.
4. Target Group showing both targets as Healthy.
5. Application Load Balancer showing Active state.
6. ALB listener forwarding traffic to `ALB-TG`.
7. Browser displaying Server 1 response.
8. Browser displaying Server 2 response.
