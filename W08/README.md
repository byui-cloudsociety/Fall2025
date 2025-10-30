# Week 08 - IAM & Security Lab (Modified for AWS Learner Labs)

Welcome to the final technical lab before the Cloud-a-thon! This week we'll master AWS Identity and Access Management (IAM) and implement comprehensive security best practices within the constraints of AWS Learner Labs.

## AWS Learner Lab Limitations
- **Cannot create new IAM roles** - We'll work with existing roles
- **Cannot modify trust relationships** - We'll use pre-configured roles
- **Can create users, groups, and policies** - We'll focus on these
- **Limited CloudTrail access** - May be pre-configured
- **Cannot enable MFA on root account** - Will practice with IAM users only

## Learning Objectives
By the end of this lab, you will be able to:
- Create and manage IAM users, groups, and policies
- Design and implement least-privilege access policies
- Configure Multi-Factor Authentication (MFA) for IAM users
- Use AWS CLI with programmatic access
- Work with existing IAM roles in Learner Labs
- Understand security monitoring with CloudTrail
- Implement security best practices for the Cloud-a-thon
- Navigate IAM within Learner Lab constraints

## Cost Awareness
- **IAM**: Completely free
- **CloudTrail**: Pre-configured in Learner Labs
- **CloudWatch**: Free tier includes basic monitoring
- We'll use less than **$0.50** of your budget today
- **Focus**: Security configuration, not expensive resources

## Lab Instructions

### Part 1: Understanding Learner Lab IAM Environment

1. **Explore Existing Roles**
   - Navigate to IAM Console
   - Go to Roles
   - Look for these pre-existing roles:
     - `LabRole` - Used for Lambda and other services
     - `EMR_EC2_DefaultRole` - For EC2 instances
     - Any other existing roles
   - Click on each role to understand their policies

2. **Document Available Roles**
   ```bash
   # On your EC2 instance or CloudShell
   aws iam list-roles --query "Roles[?starts_with(RoleName, 'Lab') || starts_with(RoleName, 'EMR')].RoleName" --output table
   ```

### Part 2: IAM Users and Groups (Full Access)

1. **Create Your First IAM User**
   - Go to Users → Create user
   - **User name**: `cloud-lab-developer`
   - **Provide user access to AWS Management Console**: ✓
   - **Console password**: Custom password → `CloudLab2025!Dev`
   - **Users must create a new password at next sign-in**: Uncheck
   - Next

2. **Create Developer Group**
   - **Permissions options**: Add user to group
   - Click "Create group"
   - **Group name**: `Developers`
   - **Attach permissions policies**: Search and select:
     - `AmazonEC2ReadOnlyAccess`
     - `AmazonS3ReadOnlyAccess`
     - `AWSLambda_ReadOnlyAccess`
   - Create group
   - Select the Developers group
   - Next → Create user

3. **Create Admin User**
   - Create another user: `cloud-lab-admin`
   - **Password**: `CloudLab2025!Admin`
   - Create new group: `Administrators`
   - Attach policy: `PowerUserAccess` (since AdministratorAccess may be restricted)
   - Create user

4. **Test User Access**
   - Copy the console sign-in URL
   - Open incognito/private browser window
   - Sign in as `cloud-lab-developer`
   - Try to:
     - View EC2 instances (✓ should work)
     - Launch an instance (✗ should fail)
     - View S3 buckets (✓ should work)
     - Create S3 bucket (✗ should fail)

### Part 3: Custom IAM Policies (Full Access)

1. **Create S3 Bucket-Specific Policy**
   - Policies → Create policy → JSON tab
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "ListAllBuckets",
         "Effect": "Allow",
         "Action": "s3:ListAllMyBuckets",
         "Resource": "*"
       },
       {
         "Sid": "AllowSpecificBucket",
         "Effect": "Allow",
         "Action": [
           "s3:GetObject",
           "s3:PutObject",
           "s3:DeleteObject",
           "s3:ListBucket"
         ],
         "Resource": [
           "arn:aws:s3:::cloudlab-*",
           "arn:aws:s3:::cloudlab-*/*"
         ]
       }
     ]
   }
   ```
   - **Policy name**: `CloudLabS3Policy`
   - **Description**: `Allow access to cloudlab-* buckets only`
   - Create policy

2. **Create Lambda Developer Policy**
   - Create policy → Visual editor
   - **Service**: Lambda
   - **Actions**:
     - List: All List actions
     - Read: All Read actions  
     - Write: InvokeFunction, UpdateFunctionCode
   - **Resources**: All resources
   - **Policy name**: `CloudLabLambdaDeveloper`
   - Create policy

3. **Create EC2 Start/Stop Policy**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "ec2:DescribeInstances",
           "ec2:DescribeInstanceStatus"
         ],
         "Resource": "*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "ec2:StartInstances",
           "ec2:StopInstances",
           "ec2:RebootInstances"
         ],
         "Resource": "arn:aws:ec2:*:*:instance/*",
         "Condition": {
           "StringEquals": {
             "aws:ResourceTag/Environment": "Development"
           }
         }
       }
     ]
   }
   ```
   - **Policy name**: `EC2StartStopDev`
   - Create policy

4. **Create CloudWatch Logs Policy**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "logs:CreateLogGroup",
           "logs:CreateLogStream",
           "logs:PutLogEvents",
           "logs:DescribeLogStreams",
           "logs:DescribeLogGroups"
         ],
         "Resource": "arn:aws:logs:*:*:log-group:/aws/lambda/*"
       }
     ]
   }
   ```
   - **Policy name**: `CloudWatchLogsLambda`
   - Create policy

### Part 4: Working with Existing Roles

1. **Explore LabRole Permissions**
   - Go to Roles → Search for `LabRole`
   - Click on LabRole
   - Review attached policies
   - Note the trust relationships (Lambda, etc.)

2. **Using LabRole with Lambda**
   - When creating Lambda functions, select `LabRole` as the execution role
   - This role has pre-configured permissions for:
     - CloudWatch Logs
     - VPC access
     - Basic Lambda execution

3. **Document Role Capabilities**
   ```bash
   # Get LabRole policies
   aws iam list-attached-role-policies --role-name LabRole
   
   # Get inline policies if any
   aws iam list-role-policies --role-name LabRole
   
   # Save this information for your Cloud-a-thon project
   ```

### Part 5: Multi-Factor Authentication (MFA)

1. **Enable Virtual MFA for IAM User**
   - Sign in as `cloud-lab-admin`
   - Go to Security credentials (top right menu)
   - Multi-factor authentication (MFA)
   - Assign MFA device
   - **Device name**: `admin-virtual-mfa`
   - **MFA device**: Authenticator app
   - Use Google Authenticator, Authy, or Microsoft Authenticator
   - Scan QR code and enter two consecutive codes
   - Complete setup

2. **Create MFA-Enforcement Policy**
   - Sign back in as your main account
   - IAM → Policies → Create policy → JSON
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "AllowViewAccountInfo",
         "Effect": "Allow",
         "Action": [
           "iam:GetAccountPasswordPolicy",
           "iam:ListVirtualMFADevices",
           "iam:ListUsers"
         ],
         "Resource": "*"
       },
       {
         "Sid": "AllowManageOwnMFA",
         "Effect": "Allow",
         "Action": [
           "iam:CreateVirtualMFADevice",
           "iam:DeleteVirtualMFADevice",
           "iam:EnableMFADevice",
           "iam:ResyncMFADevice",
           "iam:ListMFADevices",
           "iam:DeactivateMFADevice"
         ],
         "Resource": [
           "arn:aws:iam::*:mfa/${aws:username}",
           "arn:aws:iam::*:user/${aws:username}"
         ]
       },
       {
         "Sid": "DenyAllExceptListedIfNoMFA",
         "Effect": "Deny",
         "NotAction": [
           "iam:CreateVirtualMFADevice",
           "iam:EnableMFADevice",
           "iam:GetUser",
           "iam:ListMFADevices",
           "iam:ListVirtualMFADevices",
           "iam:ResyncMFADevice",
           "sts:GetSessionToken",
           "iam:ListUsers"
         ],
         "Resource": "*",
         "Condition": {
           "BoolIfExists": {
             "aws:MultiFactorAuthPresent": "false"
           }
         }
       }
     ]
   }
   ```
   - **Policy name**: `EnforceMFAPolicy`
   - Attach to Administrators group

### Part 6: AWS CLI Configuration

1. **Create Access Keys**
   - IAM → Users → `cloud-lab-developer`
   - Security credentials tab
   - Access keys → Create access key
   - **Use case**: Command Line Interface (CLI)
   - Download .csv file

2. **Configure AWS CLI on EC2**
   ```bash
   # SSH to your EC2 instance
   ssh -i your-key.pem ec2-user@[your-ec2-ip]
   
   # Configure AWS CLI
   aws configure
   # Enter your access key ID
   # Enter your secret access key
   # Default region: us-east-1
   # Default output format: json
   
   # Test access
   aws sts get-caller-identity
   aws s3 ls
   aws ec2 describe-instances
   ```

3. **Create Named Profiles**
   ```bash
   # Configure multiple profiles
   aws configure --profile developer
   # Enter cloud-lab-developer credentials
   
   aws configure --profile admin
   # Enter cloud-lab-admin credentials
   
   # Test different profiles
   aws s3 ls --profile developer
   aws ec2 describe-instances --profile admin
   ```

### Part 7: Security Monitoring with CloudTrail

1. **Check CloudTrail Status**
   - CloudTrail may be pre-configured in Learner Labs
   - Go to CloudTrail console
   - Look for existing trails
   - If available, note the S3 bucket location

2. **View Event History**
   - CloudTrail → Event history
   - Filter by:
     - Event name: `CreateUser`
     - User name: Your IAM user
     - Time range: Last hour
   - Click events to see details

3. **Monitor Your Activities**
   ```bash
   # Generate some activity
   aws s3 ls
   aws ec2 describe-instances
   aws iam list-users
   
   # Wait 5-10 minutes, then check CloudTrail
   ```

### Part 8: Lambda Security Function

1. **Create Security Monitor Lambda**
   - Lambda → Create function
   - **Function name**: `security-monitor`
   - **Runtime**: Python 3.11
   - **Execution role**: Use existing role → `LabRole`
   
   ```python
   import json
   import boto3
   import logging
   from datetime import datetime, timedelta
   
   logger = logging.getLogger()
   logger.setLevel(logging.INFO)
   
   def lambda_handler(event, context):
       """
       Security monitoring function for Learner Lab environment
       """
       
       iam = boto3.client('iam')
       cloudwatch = boto3.client('cloudwatch')
       
       security_report = {
           'timestamp': datetime.utcnow().isoformat(),
           'checks_performed': [],
           'findings': [],
           'recommendations': []
       }
       
       try:
           # Check 1: List users without MFA
           users = iam.list_users()
           users_without_mfa = []
           
           for user in users['Users']:
               username = user['UserName']
               try:
                   mfa_devices = iam.list_mfa_devices(UserName=username)
                   if not mfa_devices['MFADevices']:
                       users_without_mfa.append(username)
               except:
                   pass  # Skip if no permission
           
           security_report['checks_performed'].append('MFA Check')
           if users_without_mfa:
               security_report['findings'].append({
                   'severity': 'MEDIUM',
                   'finding': f'Users without MFA: {", ".join(users_without_mfa)}'
               })
               security_report['recommendations'].append(
                   'Enable MFA for all users with console access'
               )
           
           # Check 2: List access keys older than 90 days
           old_keys = []
           for user in users['Users']:
               username = user['UserName']
               try:
                   keys = iam.list_access_keys(UserName=username)
                   for key in keys['AccessKeyMetadata']:
                       age = datetime.utcnow().replace(tzinfo=None) - key['CreateDate'].replace(tzinfo=None)
                       if age.days > 90:
                           old_keys.append({
                               'user': username,
                               'key_id': key['AccessKeyId'],
                               'age_days': age.days
                           })
               except:
                   pass  # Skip if no permission
           
           security_report['checks_performed'].append('Access Key Age Check')
           if old_keys:
               security_report['findings'].append({
                   'severity': 'LOW',
                   'finding': f'{len(old_keys)} access keys older than 90 days'
               })
               security_report['recommendations'].append(
                   'Rotate access keys older than 90 days'
               )
           
           # Check 3: Count custom policies
           try:
               policies = iam.list_policies(Scope='Local')
               policy_count = len(policies['Policies'])
               security_report['checks_performed'].append('Custom Policy Count')
               security_report['findings'].append({
                   'severity': 'INFO',
                   'finding': f'{policy_count} custom policies found'
               })
           except:
               pass
           
           # Check 4: List groups without policies
           try:
               groups = iam.list_groups()
               empty_groups = []
               for group in groups['Groups']:
                   group_name = group['GroupName']
                   attached = iam.list_attached_group_policies(GroupName=group_name)
                   if not attached['AttachedPolicies']:
                       empty_groups.append(group_name)
               
               if empty_groups:
                   security_report['findings'].append({
                       'severity': 'LOW',
                       'finding': f'Groups without policies: {", ".join(empty_groups)}'
                   })
           except:
               pass
           
           # Add general recommendations
           security_report['recommendations'].extend([
               'Follow principle of least privilege',
               'Regularly review IAM permissions',
               'Use IAM roles for EC2 instances instead of access keys',
               'Enable CloudTrail for audit logging',
               'Tag resources for better organization'
           ])
           
           logger.info(f"Security scan completed: {len(security_report['findings'])} findings")
           
           return {
               'statusCode': 200,
               'body': json.dumps(security_report, indent=2, default=str)
           }
           
       except Exception as e:
           logger.error(f"Security scan error: {str(e)}")
           return {
               'statusCode': 500,
               'body': json.dumps({'error': str(e)})
           }
   ```

2. **Test the Security Monitor**
   - Deploy the function
   - Create test event (empty JSON: `{}`)
   - Run test
   - Review the security report

### Part 9: Resource Tagging Strategy

Since we can't create custom roles, we'll use tags for organization:

1. **Create Tagging Policy Document**
   ```markdown
   # Cloud Lab Tagging Strategy
   
   ## Required Tags for All Resources:
   - **Environment**: Development | Testing | Production
   - **Project**: CloudLab | CloudAThon
   - **Owner**: [your-name]
   - **CostCenter**: Education
   - **CreatedBy**: [IAM-username]
   - **CreatedDate**: YYYY-MM-DD
   
   ## Example Implementation:
   - EC2 Instances: Tag with Environment=Development
   - S3 Buckets: Tag with Project=CloudLab
   - Lambda Functions: Tag with Owner=[your-name]
   ```

2. **Implement Tags on Resources**
   ```bash
   # Tag an EC2 instance
   aws ec2 create-tags \
     --resources i-xxxxx \
     --tags Key=Environment,Value=Development \
            Key=Project,Value=CloudLab \
            Key=Owner,Value=YourName
   
   # Tag an S3 bucket
   aws s3api put-bucket-tagging \
     --bucket your-bucket-name \
     --tagging 'TagSet=[{Key=Environment,Value=Development},{Key=Project,Value=CloudLab}]'
   ```

### Part 10: Cloud-a-thon Security Preparation

1. **Team Access Strategy (Without Custom Roles)**
   
   Create groups for your team:
   - **CloudAthonDevelopers**: S3, Lambda, DynamoDB read/write
   - **CloudAthonAdmins**: PowerUserAccess
   - **CloudAthonReadOnly**: ReadOnlyAccess

2. **Security Best Practices Checklist**
   ```markdown
   ## Cloud-a-thon Security Checklist
   
   ### IAM Setup
   ✓ Create separate IAM users for each team member
   ✓ Organize users into groups based on responsibilities
   ✓ Apply least-privilege principle using AWS managed policies
   ✓ Enable MFA for all users with console access
   ✓ Document which existing roles to use (LabRole for Lambda)
   
   ### Access Management
   ✓ Use access keys only when necessary
   ✓ Rotate access keys before the hackathon
   ✓ Never commit credentials to version control
   ✓ Use AWS Systems Manager Parameter Store for secrets
   
   ### Resource Organization
   ✓ Implement consistent tagging strategy
   ✓ Use CloudFormation or SAM for infrastructure as code
   ✓ Document all resources created during the event
   
   ### Monitoring
   ✓ Check CloudTrail event history regularly
   ✓ Set up CloudWatch alarms for critical metrics
   ✓ Monitor AWS Cost Explorer daily
   
   ### Emergency Procedures
   ✓ Document how to revoke compromised credentials
   ✓ Know how to contact AWS Support (if available)
   ✓ Have rollback plan for critical changes
   ```

3. **Create Helper Scripts**
   ```bash
   # Create setup script for team members
   cat > team_setup.sh << 'EOF'
   #!/bin/bash
   
   # Cloud-a-thon Team Setup Script
   echo "Setting up Cloud-a-thon environment..."
   
   # Configure AWS CLI
   echo "Configuring AWS CLI..."
   aws configure
   
   # Test access
   echo "Testing AWS access..."
   aws sts get-caller-identity
   
   # List available resources
   echo "Available Lambda execution roles:"
   aws iam list-roles --query "Roles[?contains(RoleName, 'Lab')].RoleName"
   
   echo "S3 buckets:"
   aws s3 ls
   
   echo "Lambda functions:"
   aws lambda list-functions --query "Functions[].FunctionName"
   
   echo "Setup complete!"
   EOF
   
   chmod +x team_setup.sh
   ```

## Cleanup

**Minimal cleanup needed**:
1. **Disable unused IAM users** (don't delete - you might need them)
2. **Remove access keys** you won't use for Cloud-a-thon
3. **Keep all policies and groups** for next week
4. **Document everything** for your team

## Key Takeaways

- **AWS Learner Labs** have IAM limitations but still allow comprehensive security practice
- **Work with existing roles** like LabRole instead of creating new ones
- **Focus on users, groups, and policies** which have full access
- **MFA** can be enabled for IAM users (not root)
- **Tagging** becomes crucial for resource organization without custom roles
- **Documentation** is essential when working with constraints
- **Security principles** remain the same regardless of platform limitations

## Additional Resources

- [AWS Learner Lab FAQ](https://aws.amazon.com/training/learner-lab/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Working with AWS Managed Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html)
- [AWS Tagging Best Practices](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/tagging-best-practices.html)

## Troubleshooting

**Cannot create role error?**
- Use existing LabRole for Lambda functions
- Use IAM instance profiles that already exist
- Focus on policy creation instead

**Cannot modify trust relationships?**
- Work with the existing trust relationships
- Document what services can assume each role
- Plan your architecture around available roles

**MFA setup issues?**
- Ensure time is synchronized on your device
- Try using a different authenticator app
- MFA can only be set up for IAM users, not root

**Access denied even with correct policies?**
- Check if Learner Lab has additional restrictions
- Try using AWS managed policies instead of custom ones
- Some API calls may be blocked at the account level

---

