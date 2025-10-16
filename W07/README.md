# Week 07 - VPC & Networking Lab

This week we'll master Amazon VPC (Virtual Private Cloud) and learn how to build secure, scalable network architectures in the cloud!

## Learning Objectives
By the end of this lab, you will be able to:
- Design and create custom VPCs with multiple subnets
- Configure route tables and internet gateways
- Set up NAT gateways for private subnet internet access
- Implement network security with NACLs and security groups
- Create VPC endpoints for secure AWS service access
- Build multi-tier application architectures
- Understand VPC peering and connectivity options
- Monitor network traffic and troubleshoot connectivity issues

## Cost Awareness
- **VPC itself**: Free
- **NAT Gateway**: ~$0.045/hour (~$32/month) - **We'll delete this quickly!**
- **VPC Endpoints**: ~$0.01/hour per endpoint
- **Data transfer**: Various rates
- We'll use approximately **$1-2** of your budget today
- **Critical**: Delete NAT Gateway immediately after testing!

## Lab Instructions

### Part 1: Design Your Network Architecture

**Our Goal**: Build a 3-tier web application architecture:
- **Web Tier**: Public subnets with load balancer and web servers
- **Application Tier**: Private subnets with application servers
- **Database Tier**: Private subnets with RDS (isolated from internet)

**Network Design**:
```
VPC: 10.0.0.0/16 (65,536 IP addresses)
├── Public Subnet 1:  10.0.1.0/24 (AZ-1a) - Web Tier
├── Public Subnet 2:  10.0.2.0/24 (AZ-1b) - Web Tier  
├── Private Subnet 1: 10.0.11.0/24 (AZ-1a) - App Tier
├── Private Subnet 2: 10.0.12.0/24 (AZ-1b) - App Tier
├── Private Subnet 3: 10.0.21.0/24 (AZ-1a) - DB Tier
└── Private Subnet 4: 10.0.22.0/24 (AZ-1b) - DB Tier
```

### Part 2: Create Your Custom VPC

1. **Navigate to VPC Console**
   - Search for "VPC" in AWS Console
   - Click "Create VPC"

2. **VPC Configuration**
   - **Resources to create**: VPC only
   - **Name tag**: `CloudLab-VPC-[your-name]`
   - **IPv4 CIDR block**: `10.0.0.0/16`
   - **IPv6 CIDR block**: No IPv6 CIDR block
   - **Tenancy**: Default
   - Click "Create VPC"

3. **Enable DNS Settings**
   - Select your VPC
   - Actions → Edit VPC settings
   - **Enable DNS resolution**: ✓
   - **Enable DNS hostnames**: ✓
   - Save changes

### Part 3: Create Subnets

1. **Create Public Subnet 1 (Web Tier)**
   - Go to Subnets → Create subnet
   - **VPC ID**: Select your VPC
   - **Subnet name**: `Public-Web-1a`
   - **Availability Zone**: us-east-1a
   - **IPv4 CIDR block**: `10.0.1.0/24`
   - Create subnet

2. **Create Public Subnet 2 (Web Tier)**
   - **Subnet name**: `Public-Web-1b`
   - **Availability Zone**: us-east-1b
   - **IPv4 CIDR block**: `10.0.2.0/24`
   - Create subnet

3. **Create Private Subnets (App Tier)**
   - **Subnet name**: `Private-App-1a`
   - **Availability Zone**: us-east-1a
   - **IPv4 CIDR block**: `10.0.11.0/24`
   
   - **Subnet name**: `Private-App-1b`
   - **Availability Zone**: us-east-1b
   - **IPv4 CIDR block**: `10.0.12.0/24`

4. **Create Private Subnets (Database Tier)**
   - **Subnet name**: `Private-DB-1a`
   - **Availability Zone**: us-east-1a
   - **IPv4 CIDR block**: `10.0.21.0/24`
   
   - **Subnet name**: `Private-DB-1b`
   - **Availability Zone**: us-east-1b
   - **IPv4 CIDR block**: `10.0.22.0/24`

5. **Enable Auto-assign Public IPs**
   - Select `Public-Web-1a` subnet
   - Actions → Edit subnet settings
   - **Enable auto-assign public IPv4 address**: ✓
   - Save
   - Repeat for `Public-Web-1b`

### Part 4: Configure Internet Gateway and Route Tables

1. **Create Internet Gateway**
   - Go to Internet Gateways → Create internet gateway
   - **Name tag**: `CloudLab-IGW`
   - Create internet gateway
   - Select the IGW → Actions → Attach to VPC
   - Select your VPC → Attach internet gateway

2. **Create Public Route Table**
   - Go to Route Tables → Create route table
   - **Name**: `Public-RT`
   - **VPC**: Select your VPC
   - Create route table

3. **Add Internet Route to Public Route Table**
   - Select `Public-RT`
   - Routes tab → Edit routes
   - Add route:
     - **Destination**: `0.0.0.0/0`
     - **Target**: Internet Gateway (select your IGW)
   - Save changes

4. **Associate Public Subnets with Public Route Table**
   - Select `Public-RT`
   - Subnet associations tab → Edit subnet associations
   - Select both public subnets (`Public-Web-1a` and `Public-Web-1b`)
   - Save associations

### Part 5: Create NAT Gateway for Private Subnets

1. **Create NAT Gateway**
   - Go to NAT Gateways → Create NAT gateway
   - **Name**: `CloudLab-NAT`
   - **Subnet**: `Public-Web-1a` (must be in public subnet)
   - **Connectivity type**: Public
   - **Elastic IP allocation ID**: Click "Allocate Elastic IP"
   - Create NAT gateway
   - **Wait 2-3 minutes** for it to become available

2. **Create Private Route Table**
   - Go to Route Tables → Create route table
   - **Name**: `Private-RT`
   - **VPC**: Select your VPC
   - Create route table

3. **Add NAT Gateway Route to Private Route Table**
   - Select `Private-RT`
   - Routes tab → Edit routes
   - Add route:
     - **Destination**: `0.0.0.0/0`
     - **Target**: NAT Gateway (select your NAT gateway)
   - Save changes

4. **Associate Private Subnets with Private Route Table**
   - Select `Private-RT`
   - Subnet associations tab → Edit subnet associations
   - Select all private subnets (App and DB tiers)
   - Save associations

### Part 6: Test Your Network Architecture

1. **Launch Instance in Public Subnet**
   - Go to EC2 → Launch Instance
   - **Name**: `Web-Server-Public`
   - **AMI**: Amazon Linux 2023
   - **Instance type**: t2.micro
   - **Key pair**: Use existing or create new
   - **Network settings**: Edit
     - **VPC**: Select your custom VPC
     - **Subnet**: `Public-Web-1a`
     - **Auto-assign public IP**: Enable
     - **Security group**: Create new
       - **Name**: `Web-Server-SG`
       - **Description**: Security group for web servers
       - **Rules**: SSH (22) from My IP, HTTP (80) from Anywhere
   - Launch instance

2. **Launch Instance in Private Subnet**
   - **Name**: `App-Server-Private`
   - **AMI**: Amazon Linux 2023
   - **Instance type**: t2.micro
   - **Key pair**: Same as above
   - **Network settings**: Edit
     - **VPC**: Select your custom VPC
     - **Subnet**: `Private-App-1a`
     - **Auto-assign public IP**: Disable
     - **Security group**: Create new
       - **Name**: `App-Server-SG`
       - **Description**: Security group for app servers
       - **Rules**: SSH (22) from Web-Server-SG, HTTP (80) from Web-Server-SG
   - Launch instance

3. **Test Connectivity**
   ```bash
   # Connect to public instance
   ssh -i your-key.pem ec2-user@[public-instance-public-ip]
   
   # From public instance, test internet connectivity
   ping google.com
   curl http://httpbin.org/ip
   
   # Try to connect to private instance (should work)
   ssh -i your-key.pem ec2-user@[private-instance-private-ip]
   
   # From private instance, test internet connectivity via NAT
   ping google.com
   curl http://httpbin.org/ip  # Should show NAT Gateway's public IP
   ```

### Part 7: Implement Advanced Security

1. **Create Network ACLs**
   - Go to Network ACLs → Create network ACL
   - **Name**: `Web-Tier-NACL`
   - **VPC**: Select your VPC
   - Create network ACL

2. **Configure Web Tier NACL Rules**
   - Select `Web-Tier-NACL`
   - Inbound rules tab → Edit inbound rules
   - Add rules:
     ```
     Rule #  Type        Protocol  Port Range  Source      Allow/Deny
     100     HTTP        TCP       80          0.0.0.0/0   ALLOW
     110     HTTPS       TCP       443         0.0.0.0/0   ALLOW
     120     SSH         TCP       22          0.0.0.0/0   ALLOW
     130     Custom TCP  TCP       1024-65535  0.0.0.0/0   ALLOW
     *       ALL Traffic ALL       ALL         0.0.0.0/0   DENY
     ```
   
   - Outbound rules tab → Edit outbound rules
   - Add rules:
     ```
     Rule #  Type        Protocol  Port Range  Destination Allow/Deny
     100     ALL Traffic ALL       ALL         0.0.0.0/0   ALLOW
     ```

3. **Associate NACL with Public Subnets**
   - Subnet associations tab → Edit subnet associations
   - Select both public subnets
   - Save changes

4. **Create Database Security Group**
   - Go to EC2 → Security Groups → Create security group
   - **Name**: `Database-SG`
   - **Description**: Security group for database servers
   - **VPC**: Select your VPC
   - **Inbound rules**:
     - MySQL/Aurora (3306) from App-Server-SG
   - Create security group

### Part 8: Create VPC Endpoints

1. **Create S3 VPC Endpoint (Gateway Endpoint)**
   - Go to VPC → Endpoints → Create endpoint
   - **Name**: `S3-Gateway-Endpoint`
   - **Service category**: AWS services
   - **Service name**: Search for `s3` and select the Gateway type
   - **VPC**: Select your VPC
   - **Route tables**: Select `Private-RT`
   - **Policy**: Full access
   - Create endpoint

2. **Test S3 Access from Private Instance**
   ```bash
   # From private instance
   aws s3 ls
   # This should work through the VPC endpoint (no internet required)
   ```

3. **Create EC2 VPC Endpoint (Interface Endpoint)**
   - Create endpoint
   - **Name**: `EC2-Interface-Endpoint`
   - **Service name**: Search for `ec2` and select the Interface type
   - **VPC**: Select your VPC
   - **Subnets**: Select private subnets
   - **Security groups**: Create or select one allowing HTTPS (443)
   - Create endpoint

### Part 9: Build a Three-Tier Application

1. **Set Up Web Server (Public Subnet)**
   ```bash
   # Connect to your public instance
   sudo yum update -y
   sudo yum install -y httpd
   sudo systemctl start httpd
   sudo systemctl enable httpd
   
   # Create a web page that calls the app server
   sudo tee /var/www/html/index.html << 'EOF'
   <!DOCTYPE html>
   <html>
   <head>
       <title>Three-Tier VPC Application</title>
       <style>
           body { font-family: Arial, sans-serif; margin: 40px; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; }
           .container { max-width: 800px; margin: 0 auto; background: rgba(255,255,255,0.1); padding: 30px; border-radius: 15px; backdrop-filter: blur(10px); }
           .tier { margin: 20px 0; padding: 20px; background: rgba(255,255,255,0.1); border-radius: 10px; }
           .status { padding: 10px; margin: 10px 0; border-radius: 5px; }
           .success { background: rgba(40, 167, 69, 0.3); }
           .info { background: rgba(23, 162, 184, 0.3); }
           button { background: rgba(255,255,255,0.2); color: white; border: 1px solid white; padding: 10px 20px; border-radius: 5px; cursor: pointer; }
           button:hover { background: rgba(255,255,255,0.3); }
       </style>
   </head>
   <body>
       <div class="container">
           <h1>🌐 Three-Tier VPC Architecture</h1>
           
           <div class="tier">
               <h2>🌍 Web Tier (Public Subnet)</h2>
               <div class="status success">✅ You're connected to the web server!</div>
               <p><strong>Instance ID:</strong> <span id="instance-id">Loading...</span></p>
               <p><strong>Private IP:</strong> <span id="private-ip">Loading...</span></p>
               <p><strong>Availability Zone:</strong> <span id="az">Loading...</span></p>
           </div>
           
           <div class="tier">
               <h2>⚙️ Application Tier (Private Subnet)</h2>
               <div class="status info">🔗 Click to test app server connectivity</div>
               <button onclick="testAppServer()">Test App Server</button>
               <div id="app-result"></div>
           </div>
           
           <div class="tier">
               <h2>🗄️ Database Tier (Private Subnet)</h2>
               <div class="status info">🔒 Database is isolated in private subnet</div>
               <p>Database servers have no internet access and can only be reached from the application tier.</p>
           </div>
           
           <div class="tier">
               <h2>📊 Network Information</h2>
               <p><strong>VPC CIDR:</strong> 10.0.0.0/16</p>
               <p><strong>Public Subnet:</strong> 10.0.1.0/24</p>
               <p><strong>Private Subnets:</strong> 10.0.11.0/24 (App), 10.0.21.0/24 (DB)</p>
               <p><strong>Internet Access:</strong> Internet Gateway → Public Subnet</p>
               <p><strong>Private Internet:</strong> NAT Gateway → Private Subnets</p>
           </div>
       </div>
       
       <script>
       // Get instance metadata
       fetch('http://169.254.169.254/latest/meta-data/instance-id')
           .then(response => response.text())
           .then(data => document.getElementById('instance-id').textContent = data)
           .catch(error => document.getElementById('instance-id').textContent = 'Error loading');
           
       fetch('http://169.254.169.254/latest/meta-data/local-ipv4')
           .then(response => response.text())
           .then(data => document.getElementById('private-ip').textContent = data)
           .catch(error => document.getElementById('private-ip').textContent = 'Error loading');
           
       fetch('http://169.254.169.254/latest/meta-data/placement/availability-zone')
           .then(response => response.text())
           .then(data => document.getElementById('az').textContent = data)
           .catch(error => document.getElementById('az').textContent = 'Error loading');
       
       function testAppServer() {
           document.getElementById('app-result').innerHTML = '<p>🔄 Testing connection to private app server...</p>';
           
           // In a real implementation, this would make an API call to the app server
           // For this demo, we'll simulate the response
           setTimeout(() => {
               document.getElementById('app-result').innerHTML = 
                   '<div class="status success">✅ App server responded successfully!</div>' +
                   '<p>Response from private subnet: 10.0.11.x</p>' +
                   '<p>Database connection: ✅ Connected</p>' +
                   '<p>Internet access via NAT: ✅ Working</p>';
           }, 2000);
       }
       </script>
   </body>
   </html>
   EOF
   ```

2. **Set Up Application Server (Private Subnet)**
   ```bash
   # Connect to private instance through public instance (bastion host)
   # First, copy your key to the public instance
   scp -i your-key.pem your-key.pem ec2-user@[public-ip]:~/
   
   # Then connect to public instance and from there to private
   ssh -i your-key.pem ec2-user@[public-ip]
   ssh -i your-key.pem ec2-user@[private-ip]
   
   # On the private instance
   sudo yum update -y
   sudo yum install -y python3 python3-pip
   
   # Create a simple API server
   cat > app.py << 'EOF'
   from http.server import HTTPServer, BaseHTTPRequestHandler
   import json
   import subprocess
   import socket
   
   class AppHandler(BaseHTTPRequestHandler):
       def do_GET(self):
           if self.path == '/health':
               response = {
                   'status': 'healthy',
                   'server': 'app-tier',
                   'private_ip': socket.gethostbyname(socket.gethostname()),
                   'can_reach_internet': self.test_internet(),
                   'database_status': 'connected'
               }
               
               self.send_response(200)
               self.send_header('Content-type', 'application/json')
               self.end_headers()
               self.wfile.write(json.dumps(response).encode())
           else:
               self.send_response(404)
               self.end_headers()
       
       def test_internet(self):
           try:
               result = subprocess.run(['ping', '-c', '1', 'google.com'], 
                                     capture_output=True, timeout=5)
               return result.returncode == 0
           except:
               return False
   
   if __name__ == '__main__':
       server = HTTPServer(('0.0.0.0', 8080), AppHandler)
       print("App server running on port 8080...")
       server.serve_forever()
   EOF
   
   # Run the app server
   python3 app.py &
   ```

### Part 10: Monitoring and Troubleshooting

1. **VPC Flow Logs**
   - Go to VPC → Your VPC → Flow logs tab
   - Create flow log
   - **Resource type**: VPC
   - **Destination**: Send to CloudWatch Logs
   - **Log group**: Create new → `/aws/vpc/flowlogs`
   - **IAM role**: Create new role
   - Create flow log

2. **View Flow Logs**
   - Go to CloudWatch → Log groups → `/aws/vpc/flowlogs`
   - View log streams to see network traffic

3. **Test Network Connectivity**
   ```bash
   # From public instance, test routes
   traceroute google.com
   
   # Check routing table
   route -n
   
   # Test specific ports
   telnet [private-instance-ip] 22
   nc -zv [private-instance-ip] 8080
   
   # From private instance, verify NAT routing
   curl http://httpbin.org/ip
   # Should return NAT Gateway's public IP
   ```

4. **Network Troubleshooting Tools**
   ```bash
   # Install network tools
   sudo yum install -y tcpdump nmap-ncat
   
   # Monitor network traffic
   sudo tcpdump -i eth0 -n
   
   # Test DNS resolution
   nslookup google.com
   dig google.com
   
   # Check network configuration
   ip addr show
   ip route show
   ```

## Challenge Tasks

### Easy Challenge
- Create a second web server in the other availability zone
- Set up a security group that only allows traffic between specific tiers
- Test connectivity between different subnets

### Medium Challenge
- Set up an Application Load Balancer across both public subnets
- Create a bastion host for secure access to private instances
- Implement VPC peering with another VPC
- Set up VPC endpoints for additional AWS services

### Hard Challenge
- Design and implement a multi-region VPC architecture
- Set up Site-to-Site VPN connection (simulated)
- Create custom route tables for micro-segmentation
- Implement AWS Transit Gateway architecture
- Set up cross-region VPC peering

## Cleanup

**Immediate cleanup to avoid charges**:

1. **Delete NAT Gateway FIRST** (most expensive):
   - VPC → NAT Gateways → Select your NAT Gateway
   - Actions → Delete NAT Gateway → Delete
   - **Also release the Elastic IP**: 
     - EC2 → Elastic IPs → Select the EIP → Actions → Release

2. **Terminate EC2 Instances**:
   - EC2 → Instances → Select all instances → Instance state → Terminate

3. **Delete VPC Endpoints**:
   - VPC → Endpoints → Select endpoints → Actions → Delete

4. **Delete VPC (this will clean up most resources)**:
   - VPC → Your VPCs → Select your VPC → Actions → Delete VPC
   - This will delete subnets, route tables, NACLs, etc.

5. **Delete Internet Gateway** (if not auto-deleted):
   - VPC → Internet Gateways → Select IGW → Actions → Delete

**Verify everything is deleted** to avoid unexpected charges!

## Key Takeaways

- **VPCs** provide isolated network environments in the cloud
- **Subnets** enable network segmentation across availability zones
- **Route tables** control traffic flow between subnets
- **NAT Gateways** provide internet access for private subnets
- **Security groups** act as instance-level firewalls
- **NACLs** provide subnet-level network access control
- **VPC endpoints** enable private connectivity to AWS services
- **Multi-tier architectures** improve security and scalability

## Additional Resources

- [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
- [VPC Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
- [VPC Pricing](https://aws.amazon.com/vpc/pricing/)
- [Network Security in AWS](https://docs.aws.amazon.com/whitepapers/latest/aws-security-best-practices/network-security.html)

## Troubleshooting

**Instance in private subnet can't reach internet?**
- Check NAT Gateway is in public subnet
- Verify private route table has route to NAT Gateway
- Ensure NAT Gateway has Elastic IP attached

**Can't SSH to private instance?**
- Use bastion host (public instance) as jump server
- Check security groups allow SSH between instances
- Verify network ACLs don't block traffic

**VPC endpoint not working?**
- Check route tables have endpoint routes
- Verify security groups allow HTTPS (443) for interface endpoints
- Ensure DNS resolution is enabled in VPC settings

**High NAT Gateway costs?**
- Consider NAT instances for lower traffic scenarios
- Use VPC endpoints to reduce internet traffic
- Delete NAT Gateway when not needed

---

