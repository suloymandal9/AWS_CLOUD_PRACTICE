# Project-1: AWS IAM – User and Group Management with Role-Based Access Control (RBAC)

## 1. Project Overview

**Session:** 12 – AWS IAM (Identity and Access Management)  
**Task:** Task-023  
**Project:** Create IAM users and groups, attach AWS-managed policies, and test access with different permission levels.

### Objective

The objective of this practical was to understand AWS IAM and implement permission-based access control.

Created two IAM groups with different permissions, assigned IAM users to those groups, and tested which AWS actions each user could perform.

## 2. AWS Services and Features Used

- AWS Identity and Access Management (IAM)
- IAM Users
- IAM User Groups
- AWS Managed Policies
- Amazon EC2
- AWS Management Console
- Chrome Incognito Mode

## 3. Architecture

```text
                  AWS Account
                       |
                       v
                  AWS IAM
                       |
             -------------------
             |                 |
             v                 v
       ReadOnly Group      EC2 Admin Group
             |                 |
             v                 v
       ReadOnly User       EC2 Admin User
             |                 |
             v                 v
       ReadOnlyAccess    AmazonEC2FullAccess
             |                 |
             v                 v
       View AWS Resources   Manage EC2 Resources
```

## 4. Implementation Steps

### Step 1: Open IAM Dashboard

1. Logged in to AWS Management Console using the existing administrator account.
2. Searched for IAM.
3. Opened the IAM Dashboard.
4. Navigated to User Groups.

### Step 2: Create ReadOnly IAM Group

Created the following IAM group:

**Group Name:** `Task023-ReadOnly-Group`

Attached the AWS-managed policy:

`ReadOnlyAccess`

**Purpose:**

This policy provides read-only permissions to supported AWS services.

The user can view resources but cannot modify them through the permissions granted by this policy.

### Step 3: Create EC2 Admin IAM Group

Created the second IAM group:

**Group Name:** `Task023-EC2-Admin-Group`

Attached the AWS-managed policy:

`AmazonEC2FullAccess`

**Purpose:**

This policy provides broad EC2 management permissions, such as:

- Launch EC2 instances.
- Start and stop EC2 instances.
- Reboot instances.
- Terminate instances.
- Manage EC2-related resources.

Note: This policy does not provide full administrative access to every AWS service.

### Step 4: Create ReadOnly IAM User

Created the first IAM user:

`Task023-ReadOnly-User`

Configuration:

- Enabled AWS Management Console access.
- Configured a console password.
- Added the user to `Task023-ReadOnly-Group`.

The user inherited permissions from the group's `ReadOnlyAccess` policy.

### Step 5: Create EC2 Admin IAM User

Created the second IAM user:

`Task023-EC2-Admin-User`

Configuration:

- Enabled AWS Management Console access.
- Configured a console password.
- Added the user to `Task023-EC2-Admin-Group`.

The user inherited EC2 management permissions from the group's `AmazonEC2FullAccess` policy.

### Step 6: Get IAM Sign-in URL

1. Opened IAM Dashboard.
2. Located the AWS account sign-in URL.
3. Copied the URL.
4. Used Chrome Incognito Mode to test the IAM users separately.

This allowed testing without interfering with the existing administrator session.

### Step 7: Test ReadOnly IAM User

Logged in using:

`Task023-ReadOnly-User`

Tested the following:

- Accessed the AWS Management Console.
- Opened the EC2 Dashboard.
- Viewed EC2 resources.
- Verified that the user could not perform unauthorized modification operations.

Expected permission behaviour:

| Action | Expected result |
|---|---|
| View EC2 resources | Allowed |
| View instance details | Allowed |
| Launch EC2 | Denied |
| Stop EC2 | Denied |
| Terminate EC2 | Denied |
| Create IAM users | Denied |

The ReadOnly user was able to access the EC2 Dashboard and view resources.

### Step 8: Test EC2 Admin IAM User

Logged in using:

`Task023-EC2-Admin-User`

Tested EC2 access using an existing EC2 instance named:

`checking`

Instance configuration:

- Instance type: t3.micro
- Instance state: Running
- Status checks: 3/3 passed

Verified that the EC2 Admin user could access EC2 management options, including the instance state operations.

Did not stop or terminate the instance because it was being used for the practical.

### Step 9: Compare IAM Permissions

| Operation | ReadOnly User | EC2 Admin User |
|---|---|---|
| View EC2 instances | Allowed | Allowed |
| View instance details | Allowed | Allowed |
| Launch EC2 | Denied | Allowed |
| Stop EC2 | Denied | Allowed |
| Reboot EC2 | Denied | Allowed |
| Terminate EC2 | Denied | Allowed |
| Full AWS administration | Not allowed | Not allowed |

Note: The table describes the intended permissions from the attached policies. Individual actions can also depend on resource policies, permission boundaries, SCPs, and other authorization controls.

## 5. Problems Faced and Troubleshooting

### Problem 1: EC2 instance was not visible to the ReadOnly user

**Issue:**

After logging in as `Task023-ReadOnly-User`, the EC2 Instances page displayed:

`No instances`

**Investigation:**

Checked the AWS region selected in the console.

The EC2 instance had been created in:

`US East (N. Virginia) – us-east-1`

But the ReadOnly user was viewing:

`Asia Pacific (Sydney) – ap-southeast-2`

**Root Cause:**

EC2 resources are region-specific. The user was checking the wrong region.

**Resolution:**

Changed the region to US East (N. Virginia).

After changing the region, the correct EC2 resources could be viewed.

**Learning:**

Always verify the AWS region when troubleshooting missing EC2 resources.

### Problem 2: Access Denied on AWS Health Dashboard

**Issue:**

While logged in as the EC2 Admin user, the EC2 Dashboard displayed:

`Access denied`

under Service Health.

**Investigation:**

Reviewed the attached `AmazonEC2FullAccess` policy.

**Root Cause:**

The AWS Health Dashboard requires permissions separate from EC2 management permissions.

**Learning:**

`AmazonEC2FullAccess` does not provide access to every AWS service.

Different AWS services require their own permissions.

## 6. Final Result

Successfully implemented IAM user and group management with two different permission levels.

| Activity | Result |
|---|---|
| Create ReadOnly group | Successful |
| Attach ReadOnlyAccess policy | Successful |
| Create EC2 Admin group | Successful |
| Attach AmazonEC2FullAccess policy | Successful |
| Create ReadOnly IAM user | Successful |
| Create EC2 Admin IAM user | Successful |
| Assign users to groups | Successful |
| Test ReadOnly user | Performed |
| Test EC2 Admin user | Performed |
| Compare permission levels | Completed |

## 7. Key Learnings

1. Understood AWS IAM and its purpose.
2. Learned how to create IAM users.
3. Learned how to create IAM user groups.
4. Understood AWS-managed policies.
5. Learned how users inherit permissions from groups.
6. Understood the difference between ReadOnlyAccess and AmazonEC2FullAccess.
7. Practised testing IAM users through separate console sessions.
8. Understood the importance of least-privilege access.
9. Learned to troubleshoot region-related resource visibility issues.
10. Understood that permissions for different AWS services are separate.

## 8. Conclusion

Successfully implemented AWS IAM user and group management.

Created two IAM groups with different AWS-managed policies, assigned users to the respective groups, and tested their access to AWS resources.

This practical provided hands-on experience with identity management, permission assignment, access control, and troubleshooting in AWS.
