# Week 08 - IAM & Security Lab

Welcome to the final technical lab before the Cloud-a-thon! This week we'll master AWS Identity and Access Management (IAM) and implement comprehensive security best practices.

## Learning Objectives
By the end of this lab, you will be able to:
- Create and manage IAM users, groups, and roles
- Design and implement least-privilege access policies
- Configure Multi-Factor Authentication (MFA)
- Use AWS CLI with programmatic access
- Implement cross-service role assumptions
- Set up CloudTrail for audit logging
- Configure AWS Config for compliance monitoring
- Secure your AWS resources with advanced security features
- Understand security best practices for the Cloud-a-thon

## Cost Awareness
- **IAM**: Completely free
- **CloudTrail**: Free tier includes 1 management event trail
- **AWS Config**: $0.003 per configuration item (minimal cost)
- **CloudWatch**: Free tier includes basic monitoring
- We'll use less than **$0.50** of your budget today
- **Focus**: Security configuration, not expensive resources

## Lab Instructions

### Part 1: IAM Fundamentals

1. **Navigate to IAM Console**
   - Search for "IAM" in AWS Console
   - Explore the IAM dashboard
   - Notice the security recommendations

2. **Create Your First IAM User**
   - Go to Users → Create user
   - **User name**: `cloud-lab-developer`
   - **Provide user access to AWS Management Console**: ✓
   - **Console password**: Custom password → `CloudLab2025!`
   - **Users must create a new password at next sign-in**: Uncheck
   - Next

3. **Set User Permissions**
   - **Permissions options**: Add user to group
   - Click "Create group"
   - **Group name**: `Developers`
   - **Attach permissions policies**: Search and select:
     - `AmazonEC2ReadOnlyAccess`
     - `AmazonS3ReadOnlyAccess`
   - Create group
   - Select the Developers group
   - Next → Create user

4. **Test User Access**
   - Copy the console sign-in URL
   - Open incognito/private browser window
   - Sign in as `cloud-lab-developer`
   - Try to access EC2 and S3 (should work)
   - Try to launch an instance (should fail)

### Part 2: Advanced IAM Policies

1. **Create Custom Policy for Lambda Development**
   - Go to Policies → Create policy
   - **Service**: Lambda
   - **Actions**: 
     - List: All List actions
     - Read: All Read actions
     - Write: CreateFunction, UpdateFunctionCode, UpdateFunctionConfiguration
   - **Resources**: All resources
   - Next
   - **Policy name**: `LambdaDeveloperPolicy`
   - **Description**: `Custom policy for Lambda development`
   - Create policy

2. **Create Advanced S3 Policy**
   - Create policy → JSON tab
   - Replace with this policy:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "AllowListBuckets",
         "Effect": "Allow",
         "Action": "s3:ListAllMyBuckets",
         "Resource": "*"
       },
       {
         "Sid": "AllowUserFolder",
         "Effect": "Allow",
         "Action": [
           "s3:GetObject",
           "s3:PutObject",
           "s3:DeleteObject",
           "s3:ListBucket"
         ],
         "Resource": [
           "arn:aws:s3:::cloud-lab-${aws:username}",
           "arn:aws:s3:::cloud-lab-${aws:username}/*"
         ]
       },
       {
         "Sid": "DenyDeleteBucket",
         "Effect": "Deny",
         "Action": "s3:DeleteBucket",
         "Resource": "*"
       }
     ]
   }
   ```
   - **Policy name**: `S3UserFolderPolicy`
   - Create policy

3. **Create a DevOps Group**
   - Groups → Create group
   - **Group name**: `DevOps`
   - **Attach permissions policies**:
     - `AmazonEC2FullAccess`
     - `AmazonS3FullAccess`
     - `AWSLambdaFullAccess`
     - `CloudWatchReadOnlyAccess`
     - Your custom `LambdaDeveloperPolicy`
   - Create group

### Part 3: IAM Roles and Cross-Service Access

1. **Create Lambda Execution Role**
   - Go to Roles → Create role
   - **Trusted entity type**: AWS service
   - **Service**: Lambda
   - Next
   - **Permissions policies**: 
     - `AWSLambdaBasicExecutionRole`
     - `AmazonS3ReadOnlyAccess`
     - `AmazonRDSReadOnlyAccess`
   - Next
   - **Role name**: `CloudLabLambdaRole`
   - **Description**: `Role for Lambda functions in Cloud Lab`
   - Create role

2. **Create EC2 Instance Role**
   - Create role
   - **Service**: EC2
   - **Permissions policies**:
     - `AmazonS3ReadOnlyAccess`
     - `CloudWatchAgentServerPolicy`
   - **Role name**: `CloudLabEC2Role`
   - Create role

3. **Create Cross-Account Assume Role (Simulation)**
   - Create role
   - **Trusted entity type**: AWS account
   - **Account ID**: Your current account ID
   - **Require MFA**: ✓
   - Next
   - **Permissions**: `ReadOnlyAccess`
   - **Role name**: `CloudLabAuditorRole`
   - Create role

4. **Test Role Assumption**
   ```bash
   # We'll test this later with AWS CLI
   ```

### Part 4: Multi-Factor Authentication (MFA)

1. **Enable MFA for Root Account**
   - Go to Account settings (top right menu)
   - Security credentials
   - Multi-factor authentication (MFA)
   - Assign MFA device
   - **Device name**: `root-mfa`
   - **MFA device**: Authenticator app
   - Follow setup instructions with Google Authenticator or similar app

2. **Enable MFA for IAM User**
   - Go to Users → `cloud-lab-developer`
   - Security credentials tab
   - Multi-factor authentication (MFA) → Assign MFA device
   - **Device name**: `developer-mfa`
   - **MFA device**: Authenticator app
   - Complete setup

3. **Create MFA-Required Policy**
   - Policies → Create policy → JSON
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "AllowViewAccountInfo",
         "Effect": "Allow",
         "Action": [
           "iam:GetAccountPasswordPolicy",
           "iam:ListVirtualMFADevices"
         ],
         "Resource": "*"
       },
       {
         "Sid": "AllowManageOwnPasswords",
         "Effect": "Allow",
         "Action": [
           "iam:ChangePassword",
           "iam:GetUser"
         ],
         "Resource": "arn:aws:iam::*:user/${aws:username}"
       },
       {
         "Sid": "AllowManageOwnMFA",
         "Effect": "Allow",
         "Action": [
           "iam:CreateVirtualMFADevice",
           "iam:DeleteVirtualMFADevice",
           "iam:EnableMFADevice",
           "iam:ListMFADevices",
           "iam:ResyncMFADevice"
         ],
         "Resource": [
           "arn:aws:iam::*:mfa/${aws:username}",
           "arn:aws:iam::*:user/${aws:username}"
         ]
       },
       {
         "Sid": "DenyAllExceptUnlessSignedInWithMFA",
         "Effect": "Deny",
         "NotAction": [
           "iam:CreateVirtualMFADevice",
           "iam:EnableMFADevice",
           "iam:GetUser",
           "iam:ListMFADevices",
           "iam:ListVirtualMFADevices",
           "iam:ResyncMFADevice",
           "sts:GetSessionToken"
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
   - **Policy name**: `RequireMFAPolicy`
   - Create policy

### Part 5: AWS CLI and Programmatic Access

1. **Create Access Keys for Developer User**
   - Users → `cloud-lab-developer`
   - Security credentials tab
   - Access keys → Create access key
   - **Use case**: Command Line Interface (CLI)
   - Check acknowledgment box
   - Next → Create access key
   - **Download .csv file** or copy keys securely

2. **Configure AWS CLI on EC2**
   ```bash
   # Connect to your EC2 instance
   ssh -i your-key.pem ec2-user@[your-ec2-ip]
   
   # Configure AWS CLI
   aws configure
   # Access Key ID: [from step above]
   # Secret Access Key: [from step above]
   # Default region: us-east-1
   # Default output format: json
   
   # Test basic access
   aws sts get-caller-identity
   aws s3 ls
   aws ec2 describe-instances --region us-east-1
   ```

3. **Test Role Assumption**
   ```bash
   # Assume the Lambda execution role
   aws sts assume-role \
     --role-arn arn:aws:iam::ACCOUNT-ID:role/CloudLabLambdaRole \
     --role-session-name test-session
   
   # If successful, you'll get temporary credentials
   # Export them and test access
   export AWS_ACCESS_KEY_ID=[temporary-key]
   export AWS_SECRET_ACCESS_KEY=[temporary-secret]
   export AWS_SESSION_TOKEN=[session-token]
   
   # Test with new credentials
   aws sts get-caller-identity
   ```

### Part 6: CloudTrail Audit Logging

1. **Create CloudTrail**
   - Search for "CloudTrail" in console
   - Create trail
   - **Trail name**: `CloudLabAuditTrail`
   - **Apply trail to all regions**: ✓
   - **Enable CloudWatch Logs**: ✓
   - **Log group**: New → `CloudTrail/CloudLab`
   - **IAM role**: New → `CloudTrailRole`
   - **S3 bucket**: Create new bucket → `cloudtrail-logs-[your-name]-[random]`
   - Next → Create trail

2. **Generate Some Activity**
   ```bash
   # Perform various actions to generate logs
   aws s3 ls
   aws ec2 describe-instances
   aws iam list-users
   aws lambda list-functions
   ```

3. **View CloudTrail Logs**
   - CloudTrail → Event history
   - Filter by time range and user
   - Click on events to see detailed JSON
   - Go to CloudWatch → Log groups → CloudTrail/CloudLab
   - View log streams and search for specific events

### Part 7: AWS Config for Compliance

1. **Set Up AWS Config**
   - Search for "Config" in console
   - Get started
   - **Record all resources**: ✓
   - **Include global resource types**: ✓
   - **S3 bucket**: Create new → `aws-config-[your-name]-[random]`
   - **SNS topic**: Create new → `config-topic`
   - **IAM role**: Create new → `aws-config-role`
   - Next → Confirm

2. **Add Compliance Rules**
   - Rules → Add rule
   - Search for and add these rules:
     - `s3-bucket-public-read-prohibited`
     - `ec2-security-group-attached-to-eni`
     - `iam-password-policy`
     - `root-access-key-check`
   - Configure each rule with default settings

3. **Check Compliance Status**
   - Wait 10-15 minutes for initial evaluation
   - View compliance dashboard
   - Click on non-compliant resources to see details
   - Remediate issues if any

### Part 8: Security Best Practices Implementation

1. **Create Security Monitoring Lambda**
   - Go to Lambda → Create function
   - **Function name**: `security-monitor`
   - **Runtime**: Python 3.11
   - **Execution role**: Use existing role → `CloudLabLambdaRole`
   
   ```python
   import json
   import boto3
   import logging
   from datetime import datetime
   
   logger = logging.getLogger()
   logger.setLevel(logging.INFO)
   
   def lambda_handler(event, context):
       """
       Security monitoring function that checks for:
       - Root account usage
       - Failed login attempts
       - Unusual API calls
       """
       
       # Initialize clients
       iam = boto3.client('iam')
       cloudtrail = boto3.client('cloudtrail')
       
       security_alerts = []
       
       try:
           # Check for root account usage
           response = cloudtrail.lookup_events(
               LookupAttributes=[
                   {
                       'AttributeKey': 'Username',
                       'AttributeValue': 'root'
                   }
               ],
               MaxItems=10
           )
           
           root_events = response.get('Events', [])
           if root_events:
               security_alerts.append({
                   'type': 'ROOT_ACCOUNT_USAGE',
                   'severity': 'HIGH',
                   'message': f'Root account used {len(root_events)} times recently',
                   'events': len(root_events)
               })
           
           # Check for users without MFA
           users = iam.list_users()
           users_without_mfa = []
           
           for user in users['Users']:
               username = user['UserName']
               mfa_devices = iam.list_mfa_devices(UserName=username)
               
               if not mfa_devices['MFADevices']:
                   users_without_mfa.append(username)
           
           if users_without_mfa:
               security_alerts.append({
                   'type': 'USERS_WITHOUT_MFA',
                   'severity': 'MEDIUM',
                   'message': f'{len(users_without_mfa)} users without MFA',
                   'users': users_without_mfa
               })
           
           # Check for overly permissive policies
           policies = iam.list_policies(Scope='Local', MaxItems=50)
           risky_policies = []
           
           for policy in policies['Policies']:
               policy_name = policy['PolicyName']
               if 'FullAccess' in policy_name or '*' in policy_name:
                   risky_policies.append(policy_name)
           
           if risky_policies:
               security_alerts.append({
                   'type': 'RISKY_POLICIES',
                   'severity': 'MEDIUM',
                   'message': f'{len(risky_policies)} potentially risky policies found',
                   'policies': risky_policies
               })
           
           # Create security report
           security_report = {
               'timestamp': datetime.utcnow().isoformat(),
               'alert_count': len(security_alerts),
               'alerts': security_alerts,
               'recommendations': [
                   'Enable MFA for all users',
                   'Avoid using root account for daily tasks',
                   'Follow principle of least privilege',
                   'Regularly review and rotate access keys',
                   'Enable CloudTrail in all regions'
               ]
           }
           
           logger.info(f"Security scan completed. Found {len(security_alerts)} alerts.")
           
           return {
               'statusCode': 200,
               'body': json.dumps(security_report, indent=2)
           }
           
       except Exception as e:
           logger.error(f"Security monitoring error: {str(e)}")
           return {
               'statusCode': 500,
               'body': json.dumps({'error': str(e)})
           }
   ```

2. **Test Security Monitor**
   - Deploy the function
   - Test with empty event
   - Review the security report output

3. **Create IAM Policy Review Script**
   ```bash
   # Create script on EC2
   cat > security_review.sh << 'EOF'
   #!/bin/bash
   echo "=== AWS Security Review ==="
   echo "Date: $(date)"
   echo
   
   echo "1. IAM Users:"
   aws iam list-users --query 'Users[*].[UserName,CreateDate]' --output table
   echo
   
   echo "2. Users without MFA:"
   for user in $(aws iam list-users --query 'Users[*].UserName' --output text); do
       mfa_count=$(aws iam list-mfa-devices --user-name $user --query 'length(MFADevices)')
       if [ "$mfa_count" -eq 0 ]; then
           echo "  - $user (NO MFA)"
       fi
   done
   echo
   
   echo "3. Access Keys older than 90 days:"
   aws iam list-access-keys --query 'AccessKeyMetadata[?CreateDate<`2024-06-01`].[UserName,AccessKeyId,CreateDate]' --output table
   echo
   
   echo "4. Overly permissive policies:"
   aws iam list-policies --scope Local --query 'Policies[?contains(PolicyName, `FullAccess`) || contains(PolicyName, `Admin`)].[PolicyName,CreateDate]' --output table
   echo
   
   echo "5. Recent CloudTrail events:"
   aws logs describe-log-groups --log-group-name-prefix "CloudTrail" --query 'logGroups[*].logGroupName' --output table
   echo
   
   echo "=== Security Review Complete ==="
   EOF
   
   chmod +x security_review.sh
   ./security_review.sh
   ```

### Part 9: Secure Application Architecture

1. **Design Secure Multi-Service Architecture**
   - Create architecture diagram for your Cloud-a-thon project
   - Plan IAM roles for each service
   - Design least-privilege access policies

2. **Create Application-Specific Roles**
   ```bash
   # Example: Create role for web application
   aws iam create-role \
     --role-name WebAppRole \
     --assume-role-policy-document '{
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {"Service": "ec2.amazonaws.com"},
           "Action": "sts:AssumeRole"
         }
       ]
     }'
   
   # Attach minimal required policies
   aws iam attach-role-policy \
     --role-name WebAppRole \
     --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
   ```

### Part 10: Cloud-a-thon Security Preparation

1. **Create Team Access Policies**
   - Design policies for different team roles:
     - **Frontend Developer**: S3, CloudFront access
     - **Backend Developer**: Lambda, API Gateway, RDS access
     - **DevOps Engineer**: EC2, VPC, CloudWatch access
     - **Team Lead**: Read access to all resources

2. **Security Checklist for Cloud-a-thon**
   ```
   ✓ Enable MFA for all team members
   ✓ Use IAM roles instead of access keys where possible
   ✓ Follow least privilege principle
   ✓ Enable CloudTrail for audit logging
   ✓ Configure resource-based policies for S3, Lambda
   ✓ Use VPC for network isolation
   ✓ Encrypt data at rest and in transit
   ✓ Set up billing alerts to monitor costs
   ✓ Plan resource tagging strategy
   ✓ Document emergency access procedures
   ```

3. **Create Emergency Access Procedure**
   ```markdown
   # Emergency Access Procedure
   
   ## Scenario: Locked out of AWS account
   1. Contact root account owner
   2. Use root account MFA to access console
   3. Check CloudTrail for unauthorized access
   4. Reset compromised credentials
   5. Review and update security policies
   
   ## Scenario: Suspected security breach
   1. Immediately disable suspected compromised credentials
   2. Check CloudTrail for unusual activity
   3. Review AWS Config compliance status
   4. Run security monitoring Lambda function
   5. Document incident and notify team
   ```

## Cleanup

**Cost-conscious cleanup**:
1. **Delete CloudTrail trail** (to stop S3 storage costs)
2. **Delete CloudTrail S3 bucket contents**
3. **Disable AWS Config** (to stop configuration item costs)
4. **Keep IAM resources** (they're free)
5. **Delete any test Lambda functions**
6. **Release Elastic IPs if any were created**

**Keep for Cloud-a-thon**: IAM users, groups, roles, and policies you'll need next week!


## Key Takeaways

- **IAM** is the foundation of AWS security
- **Principle of least privilege** minimizes security risks
- **Roles** are preferred over access keys for AWS resources
- **MFA** significantly improves account security
- **CloudTrail** provides comprehensive audit logging
- **AWS Config** enables compliance monitoring and reporting
- **Automation** is key to maintaining security at scale
- **Defense in depth** uses multiple security layers

## Additional Resources

- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS Security Best Practices](https://aws.amazon.com/security/security-learning/)
- [CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)
- [AWS Config Developer Guide](https://docs.aws.amazon.com/config/latest/developerguide/)
- [AWS Well-Architected Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)

## Troubleshooting

**Access denied errors after creating policies?**
- Wait 1-2 minutes for IAM changes to propagate
- Check policy syntax and resource ARNs
- Verify the user/role has the policy attached

**MFA not working?**
- Ensure time is synchronized on your device
- Try generating a new code
- Check that the MFA device is properly activated

**CloudTrail events not appearing?**
- Wait 15-20 minutes for events to appear
- Check that the trail is enabled and in the correct region
- Verify S3 bucket permissions for CloudTrail

**AWS CLI assume-role failing?**
- Check that your user has sts:AssumeRole permission
- Verify the role trust policy includes your user/role
- Ensure MFA requirements are met if configured

**Config rules showing non-compliant?**
- Some rules may take time to evaluate
- Check rule parameters and scope
- Review resource configuration for compliance requirements

---