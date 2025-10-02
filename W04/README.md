# Week 04 - S3 & Static Websites Lab

This week we'll dive into Amazon S3 (Simple Storage Service) and learn how to host static websites directly from the cloud!

## Learning Objectives

By the end of this lab, you will be able to:

- Create and configure S3 buckets
- Upload and manage files in S3
- Understand S3 storage classes and pricing
- Host a static website using S3
- Configure bucket policies for public access
- Use S3 versioning and lifecycle policies
- Connect S3 with your EC2 instance from last week

## Cost Awareness

- **S3 Free Tier**: 5 GB storage, 20,000 GET requests, 2,000 PUT requests/month
- **Storage cost**: ~$0.023 per GB/month for Standard storage
- We'll use less than $1 of your budget today
- **Remember**: Delete test files after lab to save space

## Lab Instructions

### Part 1: Create Your First S3 Bucket

1. **Navigate to S3**
   - In AWS Console, search for "S3"
   - Click "Create bucket"

2. **Bucket Configuration**
   - **Bucket name**: `cloud-lab-[your-name]-[random-numbers]`
     - Example: `cloud-lab-john-12345`
     - Note: Bucket names must be globally unique!
   - **Region**: US East (N. Virginia) us-east-1
   - **Block Public Access**: Keep all checkboxes CHECKED for now
   - Leave other settings as default
   - Click "Create bucket"

3. **Upload Your First File**
   - Click on your bucket name
   - Click "Upload"
   - Add a simple text file (create one if needed)
   - Click "Upload"

### Part 2: Explore S3 Features

1. **Storage Classes**
   - Click on your uploaded file
   - Go to "Properties" tab
   - Note the current storage class (Standard)
   - Click "Edit" and explore other storage classes:
     - Standard-IA (Infrequent Access)
     - Glacier Instant Retrieval
     - Glacier Flexible Retrieval

2. **Enable Versioning**
   - Go back to bucket main page
   - Click "Properties" tab
   - Find "Bucket Versioning" section
   - Click "Edit" → Enable → Save changes

3. **Test Versioning**
   - Upload the same filename again with different content
   - Click on the file and see multiple versions
   - Try downloading different versions

### Part 3: Build a Static Website

1. **Create Website Files**
   Create these files on your computer:

   **index.html:**

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>My Cloud Portfolio</title>
       <link rel="stylesheet" href="style.css">
   </head>
   <body>
       <header>
           <h1>Welcome to My Cloud Journey</h1>
           <nav>
               <a href="#about">About</a>
               <a href="#projects">Projects</a>
               <a href="#contact">Contact</a>
           </nav>
       </header>
       
       <main>
           <section id="about">
               <h2>About Me</h2>
               <p>I'm learning cloud computing with the BYU-I Cloud Society!</p>
               <img src="cloud-icon.png" alt="Cloud Computing" width="200">
           </section>
           
           <section id="projects">
               <h2>My AWS Projects</h2>
               <ul>
                   <li>Week 3: Launched my first EC2 instance</li>
                   <li>Week 4: Built this website on S3!</li>
                   <li>Coming soon: Serverless functions with Lambda</li>
               </ul>
           </section>
           
           <section id="contact">
               <h2>Contact</h2>
               <p>Connect with me: [your-email]@byui.edu</p>
           </section>
       </main>
       
       <footer>
           <p>Hosted on Amazon S3 | Built with ❤️ and ☁️</p>
       </footer>
   </body>
   </html>
   ```

   **style.css:**

   ```css
   * {
       margin: 0;
       padding: 0;
       box-sizing: border-box;
   }
   
   body {
       font-family: 'Arial', sans-serif;
       line-height: 1.6;
       color: #333;
       background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
       min-height: 100vh;
   }
   
   header {
       background: rgba(255,255,255,0.1);
       backdrop-filter: blur(10px);
       padding: 1rem 0;
       text-align: center;
       margin-bottom: 2rem;
   }
   
   header h1 {
       color: white;
       margin-bottom: 1rem;
       font-size: 2.5rem;
   }
   
   nav a {
       color: white;
       text-decoration: none;
       margin: 0 1rem;
       padding: 0.5rem 1rem;
       border-radius: 20px;
       background: rgba(255,255,255,0.2);
       transition: all 0.3s;
   }
   
   nav a:hover {
       background: rgba(255,255,255,0.3);
       transform: translateY(-2px);
   }
   
   main {
       max-width: 800px;
       margin: 0 auto;
       padding: 0 1rem;
   }
   
   section {
       background: white;
       margin-bottom: 2rem;
       padding: 2rem;
       border-radius: 15px;
       box-shadow: 0 10px 30px rgba(0,0,0,0.1);
   }
   
   h2 {
       color: #667eea;
       margin-bottom: 1rem;
       font-size: 1.8rem;
   }
   
   ul {
       margin-left: 1.5rem;
   }
   
   li {
       margin-bottom: 0.5rem;
   }
   
   footer {
       text-align: center;
       padding: 2rem;
       color: white;
       background: rgba(0,0,0,0.2);
       margin-top: 2rem;
   }
   
   img {
       display: block;
       margin: 1rem auto;
       border-radius: 10px;
   }
   ```

   **error.html:**

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Page Not Found</title>
       <link rel="stylesheet" href="style.css">
   </head>
   <body>
       <main>
           <section style="text-align: center;">
               <h1>404 - Page Not Found</h1>
               <p>The page you're looking for doesn't exist.</p>
               <a href="index.html">← Back to Home</a>
           </section>
       </main>
   </body>
   </html>
   ```

2. **Find a Cloud Icon**
   - Search for a free cloud computing icon online (PNG format)
   - Download and save as "cloud-icon.png"

3. **Upload Website Files**
   - Upload all files (index.html, style.css, error.html, cloud-icon.png) to your S3 bucket
   - Keep default settings for now

### Part 4: Configure Static Website Hosting

1. **Enable Static Website Hosting**
   - Go to bucket Properties tab
   - Scroll to "Static website hosting"
   - Click "Edit"
   - Enable static website hosting
   - **Index document**: `index.html`
   - **Error document**: `error.html`
   - Save changes
   - **Copy the bucket website endpoint URL!**

2. **Configure Public Access**

   - Go to "Permissions" tab
   - Click "Edit" on "Block public access"
   - **UNCHECK** "Block all public access"
   - Type "confirm" and save
   - **Warning**: This makes your bucket publicly accessible

3. **Add Bucket Policy**
   - Still in Permissions tab, scroll to "Bucket policy"
   - Click "Edit" and paste this policy (replace YOUR-BUCKET-NAME):

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
       }
     ]
   }
   ```

4. **Test Your Website**
   - Visit your bucket website endpoint URL
   - Your website should be live! 🎉

### Part 5: Connect S3 with EC2

1. **Start your EC2 instance from last week**
   - Go to EC2 console
   - Start your stopped instance
   - Connect via SSH

2. **Install AWS CLI on EC2**

   ```bash
   # AWS CLI is pre-installed on Amazon Linux 2023
   aws --version
   
   # Configure AWS CLI (use your learner lab credentials)
   aws configure
   # Access Key ID: [from learner lab]
   # Secret Access Key: [from learner lab]
   # Default region: us-east-1
   # Default output format: json
   ```

3. **Download from S3 to EC2**

   ```bash
   # List your buckets
   aws s3 ls
   
   # List files in your bucket
   aws s3 ls s3://your-bucket-name/
   
   # Copy your website files to EC2
   aws s3 cp s3://your-bucket-name/index.html .
   
   # Copy entire website
   aws s3 sync s3://your-bucket-name/ ./my-website/
   
   # List downloaded files
   ls -la
   ```

4. **Upload from EC2 to S3**

   ```bash
   # Create a new file
   echo "Hello from EC2! $(date)" > ec2-message.txt
   
   # Upload to S3
   aws s3 cp ec2-message.txt s3://your-bucket-name/
   
   # Verify upload
   aws s3 ls s3://your-bucket-name/
   ```

### Part 6: Advanced S3 Features

1. **Lifecycle Management**
   - In S3 console, go to your bucket
   - Click "Management" tab
   - Create lifecycle rule:
     - Name: `cleanup-old-files`
     - Rule scope: Apply to all objects
     - Lifecycle rule actions:
       - Transition to Standard-IA after 30 days
       - Delete objects after 90 days

2. **S3 Transfer Acceleration**
   - Go to Properties tab
   - Find "Transfer acceleration"
   - Click "Edit" → Enable → Save
   - Note the accelerated endpoint URL

## Challenge

- Use S3 event notifications to trigger something when files are uploaded

## Cleanup

**Important cost-saving steps**:

1. **Delete large files** you don't need
2. **Keep your website files** (small size, negligible cost)
3. **Stop your EC2 instance** when done
4. **Remember**: Small files cost almost nothing, but good practice to clean up

## Key Takeaways

- **S3** provides unlimited, durable object storage
- **Static websites** can be hosted directly from S3 (no servers needed!)
- **Bucket policies** control access to your data
- **Storage classes** optimize costs based on access patterns
- **S3 + EC2** integration enables powerful cloud architectures

## Additional Resources

- [S3 User Guide](https://docs.aws.amazon.com/s3/latest/userguide/)
- [Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [S3 Pricing Calculator](https://calculator.aws/)

## Troubleshooting

**Website not loading?**

- Check bucket policy is correct and applied
- Verify "Block public access" is disabled
- Ensure index.html exists and is named correctly

**403 Forbidden Error?**

- Double-check bucket policy syntax
- Verify bucket name in policy matches actual bucket name
- Make sure objects have public read permissions

**AWS CLI not working on EC2?**

- Verify credentials are configured: `aws configure list`
- Check IAM permissions in learner lab
- Try using `aws sts get-caller-identity` to test connection

---

**Next Week Preview**: We'll explore serverless computing with AWS Lambda and create functions that respond to events!
