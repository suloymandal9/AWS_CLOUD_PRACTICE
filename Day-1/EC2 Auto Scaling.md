# Project-3: AWS EC2 Auto Scaling – Dynamic Scaling Using CPU Utilization

## 1. Project Overview

**Session:** 11 – EC2 Scalability & AMI  
**Task:** Task-022  
**Project:** Configure an AWS Auto Scaling Group with minimum 1, desired 2, and maximum 3 EC2 instances.

### Objective

The objective of this practical was to configure an AWS EC2 Auto Scaling Group and test its scaling behaviour based on CPU utilization.

When CPU utilization increases, AWS should automatically launch additional EC2 instances (Scale-Out).

When CPU utilization decreases, AWS can terminate unnecessary EC2 instances (Scale-In).

## 2. AWS Services Used

- Amazon EC2
- Amazon Machine Image (AMI)
- EC2 Launch Template
- EC2 Auto Scaling Group
- Amazon CloudWatch
- Target Tracking Scaling Policy
- MobaXterm
- Amazon Linux 2023
- stress-ng

## 3. Architecture

```text
                 AWS Cloud
                     |
                     v
              Custom AMI
                     |
                     v
              Launch Template
                     |
                     v
             Auto Scaling Group
                     |
          -----------------------
          |          |          |
          v          v          v
       EC2-1      EC2-2      EC2-3
          |          |          |
          -----------------------
                     |
                     v
             CloudWatch Metrics
                     |
                     v
          Target Tracking Policy
             CPU Target: 40%
                     |
          -----------------------
          |                     |
          v                     v
       Scale-Out             Scale-In
     Launch EC2             Terminate EC2
```

## 4. Implementation Steps

### Step 1: Verify Custom AMI

- Open AWS Console.
- Navigate to EC2 → AMIs.
- Verify the custom AMI created in Task-021.
- Confirm its status is Available.

### Step 2: Create Launch Template

Created an EC2 Launch Template with the following configuration:

| Configuration | Value |
|---|---|
| Launch Template | Linux-Launch-Template |
| AMI | Custom AMI from Task-021 |
| Instance Type | t3.micro |
| Key Pair | Existing .pem key pair |
| Security Group | SSH and HTTP access |

The Launch Template acts as a blueprint for launching new EC2 instances.

### Step 3: Create Auto Scaling Group

Created an Auto Scaling Group named:

`AutoScale1`

Configured the capacity:

| Setting | Value |
|---|---:|
| Minimum Capacity | 1 |
| Desired Capacity | 2 |
| Maximum Capacity | 3 |

Selected the required VPC and Availability Zone subnets.

Initially, AWS launched two EC2 instances.

### Step 4: Configure Target Tracking Scaling Policy

Created a Target Tracking Scaling Policy.

| Setting | Value |
|---|---|
| Policy Type | Target Tracking |
| Metric | Average CPU Utilization |
| Target Value | 40% |

**Working principle:**

- When CPU utilization remains above the target, AWS can launch additional EC2 instances.
- When CPU utilization remains below the target, AWS can reduce the number of instances.
- Auto Scaling respects the configured minimum and maximum capacity.

### Step 5: Connect to EC2 Instances

Connected to the Auto Scaling Group instances using MobaXterm and the existing `.pem` private key.

Verified the operating system:

```bash
cat /etc/os-release
```

Installed the CPU stress testing utility:

```bash
sudo dnf install stress-ng -y
```

Verified installation:

```bash
stress-ng --version
```

### Step 6: Generate CPU Load

Executed the following command on the EC2 instances:

```bash
stress-ng --cpu 2 --timeout 300s
```

**Command explanation:**

- `stress-ng` – CPU stress testing utility.
- `--cpu 2` – Starts two CPU workers.
- `--timeout 300s` – Runs the test for five minutes.

### Step 7: Monitor CPU Utilization

Navigated to:

EC2 → Auto Scaling Groups → AutoScale1 → Monitoring

Observed CPU utilization through CloudWatch metrics.

During testing, CPU utilization increased to approximately 92.39%.

After the stress test stopped, CPU utilization dropped to approximately 2.52%.

### Step 8: Verify Scale-Out

Checked the Auto Scaling Group Activity History.

Observed the event:

`TargetTracking-AlarmHigh`

AWS automatically changed desired capacity from 2 to 3 and launched a third EC2 instance.

Verified in Instance Management:

- Three EC2 instances were listed.
- All three were in InService state.
- All three showed Healthy status.

**Result:** Scale-Out successfully demonstrated.

### Step 9: Prepare for Scale-In Testing

Stopped the CPU stress test.

Checked instance scale-in protection and disabled protection on the instances where it had been enabled.

The purpose was to allow Auto Scaling to terminate instances when the scaling policy detected low CPU utilization.

Note: The final successful scale-in event was not captured in the available verification screenshots.

## 5. Problems Faced and Troubleshooting

### Problem 3: Could not edit the Target Tracking policy

**Issue:**

The AWS Console did not provide the expected editing option.

**Investigation:**

Checked the Auto Scaling Group settings and available actions.

**Resolution:**

Used instance scale-in protection to protect selected instances temporarily while preparing the CPU load test. Later disabled the protection to allow scale-in.

**Learning:**

Instance scale-in protection prevents selected instances from being terminated during normal Auto Scaling scale-in.



## 6. Final Result

| Test | Result |
|---|---|
| Custom AMI verification | Successful |
| Launch Template creation | Successful |
| Auto Scaling Group creation | Successful |
| Minimum capacity = 1 | Configured |
| Desired capacity = 2 | Configured initially |
| Maximum capacity = 3 | Configured |
| CPU Target Tracking = 40% | Configured |
| CPU load generation | Successful |
| Scale-Out: 2 → 3 | Verified |
| Scale-In | Configured; final event not verified |

## 7. Key Learnings

1. Understood the purpose of an EC2 Launch Template.
2. Learned how to configure an Auto Scaling Group.
3. Understood minimum, desired, and maximum capacity.
4. Learned how Target Tracking Scaling Policies work.
5. Used CloudWatch CPU metrics to monitor instance utilization.
6. Generated CPU load using stress-ng.
7. Observed automatic EC2 instance launching through Scale-Out.
8. Learned how scale-in protection affects instance termination.
9. Used Activity History to troubleshoot Auto Scaling behaviour.

## 8. Conclusion

Successfully configured an AWS EC2 Auto Scaling Group using a custom AMI and Launch Template.

Configured CPU-based Target Tracking with a 40% target and demonstrated automatic Scale-Out from two to three EC2 instances.

Also investigated automatic Scale-In behaviour, instance termination, and scale-in protection.

This practical provided hands-on experience with EC2 scalability, CloudWatch monitoring, and automated infrastructure management.
