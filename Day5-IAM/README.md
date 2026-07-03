# Day 5 — AWS IAM (Identity & Access Management) - SAA Week 2 Day 1

Today I learned **AWS IAM** — controlling who can access what, with what permissions.

IAM is critical for security (15%+ of SAA exam) and applies to every AWS account.

---

## The Core Concept

```
IAM = Permission system for AWS account

Three Questions:
1. WHO? (Principal: user, role, service)
2. WHAT? (Action: what can they do)
3. WHERE? (Resource: what resource)

Answer all three = Permission granted

Example:
WHO: John (IAM User)
WHAT: s3:GetObject (read from S3)
WHERE: my-bucket/* (files in bucket)

Permission: John can read files in my-bucket
```

---

## IAM Users

```
User = Individual person or application

Each user has:
├─ Username
├─ Password (for console login)
├─ Access Keys (for API/CLI)
│  ├─ Access Key ID
│  └─ Secret Access Key
└─ Permissions (policies attached)

When to use:
✅ Real people (developers, admins)
✅ Applications needing long-term credentials
✅ Developers using AWS CLI/SDK

When NOT to use:
❌ Never share credentials
❌ Never put access keys in code
❌ Never commit to Git
```

### User vs Root Account

```
Root Account:
- Created when you sign up for AWS
- Full access to everything
- Email and password
- Can't restrict

IAM User:
- Created in AWS account
- Limited permissions
- Separate credentials
- Can restrict fully

Best Practice:
- Don't use root account for daily work
- Create IAM users for everyone
- Lock root account away
- Root only for account recovery
```

---

## IAM Roles

```
Role = Set of permissions (not a person)

Think of it as:
- A job title (not a person)
- Can be "assumed" (temporarily)
- Expires after time limit

Each role has:
├─ Name
├─ Trust relationship (who can assume)
└─ Permissions (what they can do)

When to use:
✅ EC2 instances accessing AWS services
✅ Lambda functions accessing other services
✅ Cross-account access
✅ Temporary elevated permissions
✅ Third-party apps needing access
```

### Assume Role

```
Assume Role = Temporarily become that role

Process:
1. User requests to assume role
2. Verify trust relationship
3. Generate temporary credentials
4. Valid for 1 hour (default)
5. Credentials expire
6. Back to original permissions

Benefits:
✅ Limited time exposure
✅ Audit trail (who assumed when)
✅ Can revoke immediately
✅ No hardcoded credentials
```

---

## Policies & Permissions

### What is a Policy?

```
Policy = Document defining permissions

Format: JSON

Structure:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}

Components:
- Effect: Allow or Deny
- Action: What can be done
- Resource: What it applies to
- Condition: When it applies (optional)
```

### Policy Types

```
Identity-Based Policies
├─ Attached to user or role
├─ "John CAN read S3"
└─ Most common

Resource-Based Policies
├─ Attached to resource
├─ "Anyone with role X CAN read this S3 bucket"
└─ Less common

Managed Policies
├─ AWS pre-made policies
├─ Example: ReadOnlyAccess
└─ Can attach multiple

Inline Policies
├─ Custom, one-off policies
├─ Attached to single user/role
└─ Less reusable
```

### Common Actions

```
EC2:
- ec2:DescribeInstances (view instances)
- ec2:RunInstances (launch)
- ec2:TerminateInstances (stop)

S3:
- s3:GetObject (read file)
- s3:PutObject (upload file)
- s3:DeleteObject (delete file)
- s3:ListBucket (list files)

RDS:
- rds:DescribeDBInstances (view databases)
- rds:ModifyDBInstance (modify)
- rds:DeleteDBInstance (delete)

DynamoDB:
- dynamodb:GetItem (read)
- dynamodb:PutItem (write)
- dynamodb:Query (query)
```

---

## ARN (Amazon Resource Name)

```
ARN = Address for AWS resources

Format:
arn:partition:service:region:account-id:resource

Examples:

S3 Bucket:
arn:aws:s3:::my-bucket
└─ (S3 doesn't include region/account)

S3 Objects:
arn:aws:s3:::my-bucket/*
└─ All objects in bucket

EC2 Instance:
arn:aws:ec2:us-east-1:123456789:instance/i-1234567
└─ Specific instance in us-east-1

IAM User:
arn:aws:iam::123456789:user/john
└─ User "john" in account

RDS Database:
arn:aws:rds:us-east-1:123456789:db:mydb
└─ Database "mydb" in us-east-1

Wildcards:
* = any
? = single character

Examples:
arn:aws:s3:::my-* = any bucket starting with "my-"
arn:aws:iam::123456789:user/* = any user
arn:aws:iam::123456789:role/* = any role
```

---

## Cross-Account Access

```
Cross-Account = Access resources in different AWS account

Use case:
├─ Development account needs read-only production access
├─ Different team's account needs to assume role
└─ Third-party needs to access your resources

How it works:

Production Account:
├─ Create role "ProductionAccess"
├─ Trust: Dev account can assume
└─ Policy: Read-only

Development Account:
├─ User in dev account
├─ Has policy: Can assume ProductionAccess
└─ When assumed, gets prod credentials

Security:
✅ Temporary credentials
✅ Audit trail (CloudTrail)
✅ ExternalId (extra validation)
✅ Explicit trust relationship
```

### ExternalId for Security

```
Without ExternalId:
Role trusts Account B to assume
But if Account B's access key leaked...
Anyone with that key can assume role (BAD!)

With ExternalId:
Role requires: Account B + ExternalId
├─ Account B itself isn't enough
├─ Need the secret ExternalId too
├─ Even if key leaked, attacker needs ExternalId
└─ Separate secrets (more secure)

Example:
Trust relationship requires:
- AWS Account: 111111111111 (Account B)
- ExternalId: super-secret-key-12345
```

---

## Security Best Practices

### Principle of Least Privilege

```
Least Privilege = Give only what's needed

Bad:
- John gets AdministratorAccess
- Can do everything (too much)
- If compromised, attacker can do everything

Good:
- John gets only:
  - s3:GetObject (read S3)
  - ec2:DescribeInstances (view EC2)
  - logs:PutLogEvents (write logs)
- Can do only what's needed
- If compromised, damage is limited

How to implement:
1. What does this user need?
2. What's the minimum permission for that?
3. Attach only those permissions
4. Review every 90 days
```

### MFA (Multi-Factor Authentication)

```
MFA = Two factors required

Factor 1: Something you know
├─ Password

Factor 2: Something you have
├─ Phone with authenticator app
├─ Hardware key (YubiKey)
└─ SMS code (less secure)

Without MFA:
- Know password → Can access
- If password leaked → Account compromised

With MFA:
- Know password + have phone → Can access
- If password leaked → Still can't access
- More secure!

Where to enable:
✅ Root account (always)
✅ Admin users (always)
✅ Developers (recommended)
✅ Read-only users (optional)
```

### Access Keys vs Passwords

```
Passwords:
- For console login (web browser)
- Can have MFA
- Can expire

Access Keys:
- For API/CLI/SDK access
- Access Key ID = username
- Secret Access Key = password
- NEVER commit to Git
- NEVER hardcode in code
- Rotate every 90 days

Best Practice:
❌ Don't use long-term access keys
✅ Use temporary credentials (assume role)
✅ Use environment variables (not hardcoded)
✅ Use AWS credentials file (~/.aws/credentials)
✅ Use IAM role on EC2/Lambda (automatic)
```

---

## Real-World Scenarios

### E-commerce Company

```
Structure:

Admin Users:
- CEO, CTO
- Full access to everything
- MFA enabled
- Rarely used

Developer Users:
- Frontend, backend, mobile
- Access to: dev EC2, dev RDS, CloudWatch
- Can't delete production resources
- MFA enabled

DevOps Users:
- Can manage infrastructure
- Create, modify, but not delete production
- Deploy applications
- MFA enabled

Data Analyst Users:
- Read-only access
- Can view: S3, RDS, DynamoDB
- Can't modify anything
- MFA enabled

Services:
- EC2 instances: Assume EC2-AppRole (read S3, write logs)
- Lambda: Assume Lambda-ProcessorRole (read SQS, write DynamoDB)
- RDS: In private subnet, accessed by app role only
```

### Startup Company

```
Few People, Multiple Roles:

Alice (Founder):
- AdministratorAccess (but limited use)
- MFA enabled

Bob (Developer):
- Developer role (dev environment)
- Can assume ProdReadOnly role (temporary, with logs)
- MFA enabled

Charlie (DevOps):
- DevOps role (infrastructure)
- Can assume ProdAccess role (temporary)
- Can't delete resources
- MFA enabled

Services:
- EC2: EC2-AppRole (minimal permissions)
- Lambda: Lambda-ProcessorRole (minimal permissions)
```

---

## SAA Exam Topics Covered

✅ **Users and Identities**
- IAM users and groups
- Root account protection
- Password policy
- Access keys

✅ **Roles and Trust**
- IAM roles
- Trust relationships
- Assume role
- Cross-account access

✅ **Policies and Permissions**
- Identity-based policies
- Resource-based policies
- Managed vs inline policies
- Policy evaluation logic

✅ **Best Practices**
- Principle of least privilege
- MFA
- Access key rotation
- Service roles for EC2/Lambda

✅ **Security**
- ExternalId
- Audit trail
- Permission boundaries
- Resource-based restrictions

---


## Terraform Implementation

### Key Resources

```hcl
- aws_iam_user: Create users
- aws_iam_role: Create roles
- aws_iam_policy: Create custom policies
- aws_iam_user_policy_attachment: Attach policy to user
- aws_iam_role_policy_attachment: Attach policy to role
- aws_iam_instance_profile: Attach role to EC2
- aws_iam_virtual_mfa_device: MFA device for user
```

### Architecture as Code

Entire IAM structure defined in Terraform:
- Reproducible (same every time)
- Versioned (track in Git)
- Automated (no manual clicking)
- Documentable

---

## Key Takeaways

1. **Users**: Long-term credentials for people/applications
2. **Roles**: Temporary credentials for services/cross-account
3. **Policies**: Define what can be done
4. **ARN**: Address for resources
5. **Trust**: Explicit permission to assume role
6. **Least Privilege**: Only necessary permissions
7. **MFA**: Two-factor authentication for security
8. **Cross-Account**: Secure delegation to other accounts
9. **ExternalId**: Extra validation for cross-account
10. **Audit**: CloudTrail tracks all actions

---

## Statistics

```
Day 5:
- IAM concepts: 6+ (users, roles, policies, trust, assume, cross-account)
- SAA topics: 10+ (users, groups, roles, policies, permissions, MFA, access keys)
- Terraform resources: 8+
- Production setup: Complete
- Security levels: Multiple (admin, developer, devops, analyst)
```

---

## Interview Talking Points

"I understand IAM architecture including users for long-term credentials, roles for temporary access, and policies for fine-grained permissions. I can implement cross-account access securely using assume role with ExternalId. I follow the principle of least privilege, enable MFA for sensitive accounts, and use infrastructure as code to manage permissions consistently."

---

---

## Conclusion

IAM is the **security foundation of AWS**:
- **Users**: Access control for people
- **Roles**: Temporary access for services
- **Policies**: Fine-grained permissions
- **MFA**: Extra security layer
- **Least Privilege**: Minimize blast radius

This day covers ~15% of SAA exam.

Together with networking (Day 32) and previous services, you now have a **complete understanding of AWS infrastructure and security**.

Next: Advanced compute topics (Day 34). 🚀

---

## Common Mistakes to Avoid

❌ Sharing AWS credentials
- Use IAM users, not root account

❌ Hardcoding access keys
- Use roles on EC2/Lambda
- Use environment variables
- Use AWS credentials file

❌ Not using MFA
- Enable for admin accounts
- Enable for anyone with access

❌ Not following least privilege
- Give minimum permissions needed
- Not full admin access

❌ Not rotating access keys
- Every 90 days
- When someone leaves

❌ Not auditing permissions
- Review quarterly
- Remove unused permissions
- Check CloudTrail

---

## Cost Impact

IAM is FREE:
- Users: $0
- Roles: $0
- Policies: $0
- MFA: $0

Even better: Better security at no cost!

---

## Resources Used

- AWS IAM documentation
- Terraform AWS provider
- AWS security best practices
- SAA study materials