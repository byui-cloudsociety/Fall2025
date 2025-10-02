# Week 03 - EC2 Fundamentals Lab

Welcome to your first hands-on AWS lab! Today you'll learn the foundation of cloud computing by launching and managing your first EC2 instance.

## Learning Objectives

By the end of this lab, you will be able to:

- Launch an EC2 instance using the AWS Console
- Connect to your instance via SSH
- Configure basic security groups
- Understand instance types and pricing
- Monitor basic instance metrics
- Properly stop and terminate instances

## Cost Awareness

- **t2.micro instances are FREE TIER eligible** (750 hours/month)
- Always STOP instances when not in use to save credits
- We'll use less than $2 of your $50 budget today

## Lab Instructions

### Part 1: Launch Your First EC2 Instance

1. **Log into AWS Console**
   - Navigate to AWS Learner Lab
   - Click "Start Lab" and wait for green dot
   - Click "AWS" to open console

2. **Navigate to EC2**
   - Search for "EC2" in the services search bar
   - Click on "EC2" to open the service

3. **Launch Instance**
   - Click "Launch Instance" button
   - **Name**: `MyFirstInstance-[YourName]`
   - **AMI**: Select "Amazon Linux 2023 AMI" (should be default)
   - **Instance Type**: `t2.micro` (Free tier eligible)
   - **Key Pair**: Click "Create new key pair"
     - Name: `my-cloud-key`
     - Type: RSA
     - Format: `.pem`
     - Download and save the key file!

4. **Configure Security Group**
   - Create new security group named: `web-server-sg`
   - Add rules:
     - SSH (port 22) - Source: My IP
     - HTTP (port 80) - Source: Anywhere (0.0.0.0/0)

5. **Storage**: Keep default (8 GB gp3)

6. **Launch Instance**
   - Review settings and click "Launch Instance"
   - Wait for instance to enter "Running" state

### Part 2: Connect to Your Instance

#### For Windows Users (PuTTY)

1. Download and install PuTTY
2. Use PuTTYgen to convert .pem to .ppk format
3. Connect using your instance's public IP

#### For Mac/Linux Users

1. Open Terminal
2. Navigate to where you downloaded the key file
3. Set correct permissions:

   ```bash
   chmod 400 my-cloud-key.pem
   ```

4. Connect to your instance:

   ```bash
   ssh -i my-cloud-key.pem ec2-user@[YOUR-INSTANCE-PUBLIC-IP]
   ```

### Part 3: Explore Your Instance

Once connected, run these commands to explore:

```bash
# Check system information
uname -a

# Check current user
whoami

# Check available disk space
df -h

# Check memory usage
free -h

# Check running processes
top
# Press 'q' to quit

# Update the system
sudo yum update -y

# Install a web server
sudo yum install -y httpd

# Start the web server
sudo systemctl start httpd
sudo systemctl enable httpd

# Create a simple web page
sudo echo "<h1>Hello from $(hostname)!</h1>" > /var/www/html/index.html
```

### Part 4: Test Your Web Server

1. Copy your instance's **Public IPv4 address** from the EC2 console
2. Open a web browser and navigate to: `http://[YOUR-INSTANCE-PUBLIC-IP]`
3. You should see "Hello from [hostname]!" displayed

### Part 5: Monitor Your Instance

1. **In EC2 Console**:
   - Select your instance
   - Click "Monitoring" tab
   - Explore CPU utilization, Network in/out metrics

2. **Generate some CPU load** (in SSH session):

   ```bash
   # This will use CPU for about 30 seconds
   dd if=/dev/zero of=/dev/null &
   sleep 30
   kill %1
   ```

3. **Check metrics again** after 5 minutes to see the CPU spike

### Part 6: Instance Management

1. **Stop your instance**:
   - Right-click instance → Instance State → Stop
   - Wait for state to change to "Stopped"
   - Note: Public IP will change when restarted

2. **Start your instance**:
   - Right-click instance → Instance State → Start
   - Note the new public IP address

3. **Create an AMI (Snapshot)**:
   - Right-click instance → Image and templates → Create image
   - Name: `my-web-server-ami`
   - Description: `Web server with httpd installed`

## Challenge

- Research and configure automatic security updates

## Cleanup

**Before leaving lab**:

1. **STOP** your instance (don't terminate yet - we'll use it next week)
2. Verify it shows "Stopped" state
3. Remember: Stopped instances don't incur compute charges!

## Key Takeaways

- **EC2 instances** are virtual servers in the cloud
- **Security Groups** act as virtual firewalls
- **Key Pairs** provide secure SSH access
- **Stop vs Terminate**: Stop preserves the instance, Terminate deletes it
- **t2.micro** instances are free tier eligible - use them for learning!

## Additional Resources

- [EC2 User Guide](https://docs.aws.amazon.com/ec2/latest/userguide/)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Free Tier Details](https://aws.amazon.com/free/)

## Troubleshooting

**Can't connect via SSH?**

- Check security group allows SSH from your IP
- Verify key file permissions (chmod 400)
- Make sure instance is in "running" state

**Web page not loading?**

- Check security group allows HTTP (port 80)
- Verify httpd service is running: `sudo systemctl status httpd`
- Try using instance's public IP, not private IP

---

**Next Week Preview**: We'll explore S3 storage and learn how to host static websites in the cloud!
