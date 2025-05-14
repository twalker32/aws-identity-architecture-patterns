# Cross-Account Access with Session Control in AWS Organizations

## 📘 Scenario Overview

A university's IT team manages multiple AWS accounts under AWS Organizations. They want a way to:

- Allow the shared IT admin account to assume roles in student project accounts
- Enforce **maximum session duration**
- Ensure access is scoped to **EC2 actions only**
- Maintain centralized visibility and governance

---

## ✅ Architecture Solution

### 1. **Use IAM Roles with Trust Policies in Student Accounts**

Create a role in the student account (`StudentScopedAccessRole`) with a trust policy like:

```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::<IT-Account-ID>:role/AdminAccess"
  },
  "Action": "sts:AssumeRole",
  "Condition": {
    "StringEquals": {
      "aws:PrincipalTag/Department": "IT"
    }
  }
}
```
### 2. **Attach Permission Policy (EC2-Only Access)**

Attach a policy to `StudentScopedAccessRole` that limits actions to specific EC2 permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    }
  ]
}
```
### 3. **Enforce Session Duration**

When creating the `StudentScopedAccessRole`, set the `MaxSessionDuration` to **3600 seconds (1 hour)**.

This setting limits how long the IT admin can assume the role in one session, helping reduce risk in case credentials are exposed or left open.

> 🔐 This is a role attribute, not part of the permission or trust policy. It must be set in the **role configuration** during creation.

You can configure this via:

- **AWS Console**: Under “Maximum CLI/API session duration” when creating the role
- **AWS CLI**:

```bash
aws iam update-role \
  --role-name StudentScopedAccessRole \
  --max-session-duration 3600
---
```
### 🔥 Why It Matters

Session limits prevent overprivileged access from lingering. It’s a lightweight governance guardrail that **reduces risk without breaking workflows**.
