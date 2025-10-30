# Week 06 - RDS & Databases Lab

This week we'll dive into Amazon RDS (Relational Database Service) and learn how to manage databases in the cloud without the operational overhead!

## AWS Learner Lab Note
In AWS Learner Labs, you have **limited IAM permissions**. When creating RDS instances or connecting Lambda to databases, you'll work within these constraints. For Lambda functions, always use the existing **`LabRole`** as the execution role.

## Learning Objectives
By the end of this lab, you will be able to:
- Create and configure RDS database instances
- Connect to RDS from EC2 instances
- Understand database security groups and networking
- Perform basic database operations (CRUD)
- Configure automated backups and snapshots
- Monitor database performance
- Connect Lambda functions to RDS
- Understand RDS pricing and optimization strategies

## Cost Awareness
- **RDS Free Tier**: 750 hours of db.t3.micro instances (db.t2.micro may not be available)
- **Storage**: 20 GB General Purpose SSD storage
- **Backups**: 20 GB backup storage
- We'll use approximately **$2-3** of your budget today
- **Critical**: Always stop/delete RDS instances when not in use!

## Lab Instructions

### Part 1: Create Your First RDS Database

1. **Navigate to RDS**
   - In AWS Console, search for "RDS"
   - Click "Create database"

2. **Database Configuration**
   - **Engine type**: MySQL
   - **Engine version**: MySQL 8.0.35 (or latest 8.0.x version available)
   - **Templates**: Free tier
   - **DB instance identifier**: `cloud-lab-db-[your-name]`
   - **Master username**: `admin`
   - **Master password**: `CloudLab2025!` (write this down!)
   - **Confirm password**: `CloudLab2025!`

3. **Instance Configuration**
   - **DB instance class**: db.t3.micro (Free tier eligible)
     - **Note**: If db.t3.micro is not available, use db.t2.micro
   - **Storage type**: General Purpose SSD (gp3)
   - **Allocated storage**: 20 GB (minimum)
   - **Enable storage autoscaling**: Uncheck for cost control

4. **Connectivity Settings**
   - **Compute resource**: Don't connect to an EC2 compute resource
   - **VPC**: Default VPC
   - **DB Subnet group**: default
   - **Public access**: **Yes** (for easier connection in lab)
   - **VPC security group**: Create new
   - **New VPC security group name**: `rds-mysql-sg`
   - **Availability Zone**: No preference

5. **Database Authentication**
   - **Database authentication**: Password authentication

6. **Additional Configuration** (click to expand)
   - **Initial database name**: `cloudlab` (Important: don't skip this!)
   - **DB parameter group**: default.mysql8.0
   - **Option group**: default:mysql-8-0
   - **Backup retention period**: 7 days
   - **Enable automated backups**: Yes
   - **Backup window**: No preference
   - **Copy tags to snapshots**: Yes
   - **Enable Enhanced monitoring**: No (to save costs)
   - **Enable Performance Insights**: No (to save costs)
   - **Enable deletion protection**: No (for easy cleanup)

7. **Create Database**
   - Review settings
   - **Estimated monthly costs** should show free tier
   - Click "Create database"
   - **Wait 5-10 minutes** for creation (Status will change from "Creating" to "Available")

### Part 2: Configure Security Groups

1. **Find Your RDS Security Group**
   - While RDS is creating, go to EC2 Console → Security Groups
   - Find the security group created (named `rds-mysql-sg` or `rds-launch-wizard-X`)

2. **Edit Inbound Rules**
   - Select the RDS security group
   - Click "Edit inbound rules"
   - You should see a rule for MySQL/Aurora (port 3306)
   - Modify the rule:
     - **Type**: MySQL/Aurora
     - **Port**: 3306
     - **Source**: My IP (this will auto-populate your current IP)
     - **Description**: My IP for database access
   - Click "Save rules"

3. **Note the Database Endpoint**
   - Go back to RDS Console
   - Click on your database instance
   - Wait until Status shows "Available"
   - Under "Connectivity & security" tab, copy the **Endpoint** 
     - It looks like: `cloud-lab-db-yourname.xxxxxxxxx.us-east-1.rds.amazonaws.com`
   - **Save this endpoint** - you'll need it frequently!

### Part 3: Connect from EC2

1. **Start Your EC2 Instance**
   - Go to EC2 Console
   - Start your instance from previous weeks (or create a new t2.micro instance if needed)
   - Connect via SSH

2. **Update Security Groups for EC2 → RDS Connection**
   - Go to EC2 Console → Security Groups
   - Find your EC2 instance's security group
   - Note the Security Group ID (e.g., sg-xxxxxxxxx)
   - Go back to your RDS security group
   - Edit inbound rules → Add rule:
     - **Type**: MySQL/Aurora
     - **Port**: 3306
     - **Source**: Custom → Select your EC2 security group
     - **Description**: Allow from EC2 instance
   - Save rules

3. **Install MySQL Client**
   ```bash
   # Update system
   sudo yum update -y
   
   # Install MySQL client
   sudo yum install -y mysql
   
   # Verify installation
   mysql --version
   ```

4. **Connect to RDS Database**
   ```bash
   # Connect to your RDS instance (replace with your endpoint)
   mysql -h cloud-lab-db-yourname.xxxxxxxxx.us-east-1.rds.amazonaws.com -u admin -p
   # Enter password: CloudLab2025!
   
   # You should see:
   # Welcome to the MySQL monitor...
   # mysql>
   ```

5. **Basic Database Operations**
   ```sql
   -- Show current database
   SELECT DATABASE();
   
   -- Show all databases
   SHOW DATABASES;
   
   -- Use the cloudlab database
   USE cloudlab;
   
   -- Create a table for student information
   CREATE TABLE students (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100) NOT NULL,
       email VARCHAR(100) UNIQUE NOT NULL,
       major VARCHAR(50),
       year INT,
       gpa DECIMAL(3,2),
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   
   -- Show table structure
   DESCRIBE students;
   
   -- Insert sample data
   INSERT INTO students (name, email, major, year, gpa) VALUES
   ('John Smith', 'john.smith@byui.edu', 'Computer Science', 3, 3.75),
   ('Sarah Johnson', 'sarah.johnson@byui.edu', 'Information Technology', 2, 3.90),
   ('Mike Wilson', 'mike.wilson@byui.edu', 'Software Engineering', 4, 3.60),
   ('Emily Davis', 'emily.davis@byui.edu', 'Computer Science', 1, 3.85);
   
   -- Query all students
   SELECT * FROM students;
   
   -- Query students by major
   SELECT name, gpa FROM students WHERE major = 'Computer Science';
   
   -- Calculate average GPA
   SELECT AVG(gpa) as average_gpa FROM students;
   
   -- Calculate average GPA by major
   SELECT major, AVG(gpa) as avg_gpa, COUNT(*) as num_students 
   FROM students 
   GROUP BY major;
   
   -- Update a student's information
   UPDATE students SET gpa = 3.80 WHERE name = 'John Smith';
   
   -- Verify update
   SELECT * FROM students WHERE name = 'John Smith';
   
   -- Exit MySQL
   EXIT;
   ```

### Part 4: Create a Web Application with Database

1. **Install Required Packages on EC2**
   ```bash
   # Install Python and pip
   sudo yum install -y python3 python3-pip
   
   # Install required Python packages
   pip3 install flask pymysql --user
   
   # Verify installation
   pip3 list | grep -E "Flask|PyMySQL"
   ```

2. **Create a Simple Web Application**
   ```bash
   # Create app directory
   mkdir ~/student-app
   cd ~/student-app
   
   # Create the Flask application
   nano app.py
   ```

   **Paste this code into app.py** (use Ctrl+O to save, Ctrl+X to exit):

   ```python
   from flask import Flask, render_template, request, redirect, url_for, flash
   import pymysql
   import os
   
   app = Flask(__name__)
   app.secret_key = 'your-secret-key-change-this-in-production'
   
   # Database configuration - REPLACE WITH YOUR ENDPOINT!
   DB_HOST = 'YOUR_RDS_ENDPOINT_HERE'  # Replace with your RDS endpoint
   DB_USER = 'admin'
   DB_PASSWORD = 'CloudLab2025!'
   DB_NAME = 'cloudlab'
   
   def get_db_connection():
       return pymysql.connect(
           host=DB_HOST,
           user=DB_USER,
           password=DB_PASSWORD,
           database=DB_NAME,
           cursorclass=pymysql.cursors.DictCursor
       )
   
   @app.route('/')
   def index():
       try:
           conn = get_db_connection()
           with conn.cursor() as cursor:
               cursor.execute("SELECT * FROM students ORDER BY name")
               students = cursor.fetchall()
           conn.close()
           return render_template('index.html', students=students)
       except Exception as e:
           flash(f'Database error: {str(e)}', 'error')
           return render_template('index.html', students=[])
   
   @app.route('/add', methods=['GET', 'POST'])
   def add_student():
       if request.method == 'POST':
           try:
               name = request.form['name']
               email = request.form['email']
               major = request.form['major']
               year = int(request.form['year'])
               gpa = float(request.form['gpa'])
               
               conn = get_db_connection()
               with conn.cursor() as cursor:
                   cursor.execute(
                       "INSERT INTO students (name, email, major, year, gpa) VALUES (%s, %s, %s, %s, %s)",
                       (name, email, major, year, gpa)
                   )
               conn.commit()
               conn.close()
               
               flash('Student added successfully!', 'success')
               return redirect(url_for('index'))
           except Exception as e:
               flash(f'Error adding student: {str(e)}', 'error')
       
       return render_template('add_student.html')
   
   @app.route('/stats')
   def stats():
       try:
           conn = get_db_connection()
           with conn.cursor() as cursor:
               # Get statistics
               cursor.execute("SELECT COUNT(*) as total_students FROM students")
               total = cursor.fetchone()['total_students']
               
               cursor.execute("SELECT AVG(gpa) as avg_gpa FROM students")
               avg_gpa = cursor.fetchone()['avg_gpa']
               
               cursor.execute("SELECT major, COUNT(*) as count, AVG(gpa) as avg_gpa FROM students GROUP BY major")
               major_stats = cursor.fetchall()
               
           conn.close()
           
           return render_template('stats.html', 
                                total=total, 
                                avg_gpa=round(avg_gpa, 2) if avg_gpa else 0,
                                major_stats=major_stats)
       except Exception as e:
           flash(f'Database error: {str(e)}', 'error')
           return render_template('stats.html', total=0, avg_gpa=0, major_stats=[])
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000, debug=True)
   ```

   **Important**: Edit the file and replace `YOUR_RDS_ENDPOINT_HERE` with your actual RDS endpoint!

3. **Create HTML Templates**
   ```bash
   # Create templates directory
   mkdir templates
   
   # Create base template
   nano templates/base.html
   ```

   **Paste this code**:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>{% block title %}Student Management{% endblock %}</title>
       <style>
           * { margin: 0; padding: 0; box-sizing: border-box; }
           body { font-family: Arial, sans-serif; margin: 40px; background: #f5f5f5; }
           .container { max-width: 1000px; margin: 0 auto; background: white; padding: 30px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
           h1 { color: #333; margin-bottom: 10px; }
           .subtitle { color: #666; margin-bottom: 30px; }
           .nav { margin-bottom: 30px; padding-bottom: 20px; border-bottom: 2px solid #eee; }
           .nav a { margin-right: 20px; text-decoration: none; color: #007bff; font-weight: bold; padding: 8px 16px; border-radius: 5px; transition: background 0.3s; }
           .nav a:hover { background: #e7f3ff; }
           .flash { padding: 15px; margin: 15px 0; border-radius: 5px; }
           .flash.success { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
           .flash.error { background: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
           table { width: 100%; border-collapse: collapse; margin-top: 20px; }
           th, td { padding: 12px; text-align: left; border-bottom: 1px solid #ddd; }
           th { background-color: #f8f9fa; font-weight: bold; color: #333; }
           tr:hover { background-color: #f8f9fa; }
           .form-group { margin-bottom: 20px; }
           label { display: block; margin-bottom: 8px; font-weight: bold; color: #333; }
           input, select { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 5px; box-sizing: border-box; font-size: 14px; }
           input:focus, select:focus { outline: none; border-color: #007bff; }
           button { background: #007bff; color: white; padding: 12px 24px; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; font-weight: bold; }
           button:hover { background: #0056b3; }
           .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; margin: 20px 0; }
           .stat-card { background: #f8f9fa; padding: 20px; border-radius: 8px; text-align: center; }
           .stat-card h3 { color: #666; font-size: 14px; margin-bottom: 10px; }
           .stat-card .value { font-size: 32px; color: #007bff; font-weight: bold; }
       </style>
   </head>
   <body>
       <div class="container">
           <h1>🎓 Student Management System</h1>
           <p class="subtitle">Powered by Amazon RDS MySQL Database</p>
           
           <div class="nav">
               <a href="/">📋 All Students</a>
               <a href="/add">➕ Add Student</a>
               <a href="/stats">📊 Statistics</a>
           </div>
           
           {% with messages = get_flashed_messages(with_categories=true) %}
               {% if messages %}
                   {% for category, message in messages %}
                       <div class="flash {{ category }}">{{ message }}</div>
                   {% endfor %}
               {% endif %}
           {% endwith %}
           
           {% block content %}{% endblock %}
       </div>
   </body>
   </html>
   ```

   **Create index.html**:
   ```bash
   nano templates/index.html
   ```

   ```html
   {% extends "base.html" %}
   
   {% block title %}Student List{% endblock %}
   
   {% block content %}
   <h2>Student Directory</h2>
   
   {% if students %}
   <p>Total students: <strong>{{ students|length }}</strong></p>
   <table>
       <thead>
           <tr>
               <th>ID</th>
               <th>Name</th>
               <th>Email</th>
               <th>Major</th>
               <th>Year</th>
               <th>GPA</th>
               <th>Created</th>
           </tr>
       </thead>
       <tbody>
           {% for student in students %}
           <tr>
               <td>{{ student.id }}</td>
               <td>{{ student.name }}</td>
               <td>{{ student.email }}</td>
               <td>{{ student.major }}</td>
               <td>{{ student.year }}</td>
               <td>{{ "%.2f"|format(student.gpa) }}</td>
               <td>{{ student.created_at.strftime('%Y-%m-%d %H:%M') }}</td>
           </tr>
           {% endfor %}
       </tbody>
   </table>
   {% else %}
   <p>No students found. <a href="/add">Add the first student!</a></p>
   {% endif %}
   {% endblock %}
   ```

   **Create add_student.html**:
   ```bash
   nano templates/add_student.html
   ```

   ```html
   {% extends "base.html" %}
   
   {% block title %}Add Student{% endblock %}
   
   {% block content %}
   <h2>Add New Student</h2>
   
   <form method="POST">
       <div class="form-group">
           <label for="name">Full Name:</label>
           <input type="text" id="name" name="name" required placeholder="John Smith">
       </div>
       
       <div class="form-group">
           <label for="email">Email Address:</label>
           <input type="email" id="email" name="email" required placeholder="john.smith@byui.edu">
       </div>
       
       <div class="form-group">
           <label for="major">Major:</label>
           <select id="major" name="major" required>
               <option value="">Select Major</option>
               <option value="Computer Science">Computer Science</option>
               <option value="Information Technology">Information Technology</option>
               <option value="Software Engineering">Software Engineering</option>
               <option value="Data Science">Data Science</option>
               <option value="Cybersecurity">Cybersecurity</option>
               <option value="Web Development">Web Development</option>
           </select>
       </div>
       
       <div class="form-group">
           <label for="year">Year:</label>
           <select id="year" name="year" required>
               <option value="">Select Year</option>
               <option value="1">Freshman (1st Year)</option>
               <option value="2">Sophomore (2nd Year)</option>
               <option value="3">Junior (3rd Year)</option>
               <option value="4">Senior (4th Year)</option>
           </select>
       </div>
       
       <div class="form-group">
           <label for="gpa">GPA (0.00 - 4.00):</label>
           <input type="number" id="gpa" name="gpa" min="0" max="4" step="0.01" required placeholder="3.75">
       </div>
       
       <button type="submit">Add Student</button>
       <a href="/" style="margin-left: 15px; color: #666;">Cancel</a>
   </form>
   {% endblock %}
   ```

   **Create stats.html**:
   ```bash
   nano templates/stats.html
   ```

   ```html
   {% extends "base.html" %}
   
   {% block title %}Statistics{% endblock %}
   
   {% block content %}
   <h2>Student Statistics</h2>
   
   <div class="stats-grid">
       <div class="stat-card">
           <h3>Total Students</h3>
           <div class="value">{{ total }}</div>
       </div>
       
       <div class="stat-card">
           <h3>Average GPA</h3>
           <div class="value" style="color: #28a745;">{{ avg_gpa }}</div>
       </div>
   </div>
   
   <h3 style="margin-top: 30px;">Students by Major</h3>
   {% if major_stats %}
   <table>
       <thead>
           <tr>
               <th>Major</th>
               <th>Number of Students</th>
               <th>Average GPA</th>
           </tr>
       </thead>
       <tbody>
           {% for stat in major_stats %}
           <tr>
               <td>{{ stat.major }}</td>
               <td>{{ stat.count }}</td>
               <td>{{ "%.2f"|format(stat.avg_gpa) }}</td>
           </tr>
           {% endfor %}
       </tbody>
   </table>
   {% else %}
   <p>No data available.</p>
   {% endif %}
   {% endblock %}
   ```

4. **Update EC2 Security Group**
   ```bash
   # First, note your EC2's public IP
   curl http://169.254.169.254/latest/meta-data/public-ipv4
   ```

   - Go to EC2 Console → Security Groups
   - Select your EC2 instance's security group
   - Edit inbound rules → Add rule:
     - **Type**: Custom TCP
     - **Port**: 5000
     - **Source**: My IP
     - **Description**: Flask application
   - Save rules

5. **Run the Web Application**
   ```bash
   # Make sure you're in the student-app directory
   cd ~/student-app
   
   # Run the Flask application
   python3 app.py
   
   # You should see:
   # * Running on http://0.0.0.0:5000
   ```

6. **Test Your Web Application**
   - Open a browser
   - Visit: `http://[your-ec2-public-ip]:5000`
   - You should see your Student Management System!
   - Try adding students and viewing statistics
   - Press Ctrl+C in terminal to stop the application

### Part 5: Connect Lambda to RDS

1. **Important Note About Lambda and RDS**
   - Lambda functions in a VPC can access RDS in private subnets
   - For this lab, we'll create a Lambda that could connect to RDS
   - **Note**: In Learner Labs, Lambda-to-RDS connections may have networking constraints

2. **Create Database Lambda Function**
   - Go to Lambda Console
   - Create function: `student-api-[your-name]`
   - **Runtime**: Python 3.13
   - **Execution role**: Use an existing role → Select `LabRole`

3. **Configure VPC Settings (if needed)**
   - Go to Configuration → VPC
   - If your RDS is in default VPC, you may need to add Lambda to VPC
   - **For this lab**: We'll keep Lambda outside VPC for simplicity
   - **In production**: Lambda should be in same VPC as RDS for security

4. **Add RDS Connection Code**

   ```python
   import json
   import os
   
   # Note: This is a demonstration. In Learner Labs, Lambda may not have
   # network access to your RDS instance unless properly configured in VPC
   
   def lambda_handler(event, context):
       """
       Database API Lambda function
       
       In production, you would:
       1. Install pymysql in a Lambda Layer
       2. Configure Lambda in same VPC as RDS
       3. Use Secrets Manager for database credentials
       4. Implement connection pooling
       """
       
       # Database configuration (use environment variables in production!)
       DB_CONFIG = {
           'host': os.environ.get('DB_HOST', 'your-rds-endpoint'),
           'user': os.environ.get('DB_USER', 'admin'),
           'password': os.environ.get('DB_PASSWORD', 'CloudLab2025!'),
           'database': os.environ.get('DB_NAME', 'cloudlab')
       }
       
       http_method = event.get('httpMethod', 'GET')
       
       # Simulated response (actual database connection would require pymysql layer)
       if http_method == 'GET':
           return {
               'statusCode': 200,
               'headers': {
                   'Content-Type': 'application/json',
                   'Access-Control-Allow-Origin': '*'
               },
               'body': json.dumps({
                   'message': 'Database API endpoint',
                   'note': 'In production, this would query the RDS database',
                   'database_config': {
                       'host': DB_CONFIG['host'],
                       'database': DB_CONFIG['database'],
                       'user': DB_CONFIG['user']
                   },
                   'instructions': [
                       'To actually connect to RDS from Lambda:',
                       '1. Add Lambda to same VPC as RDS',
                       '2. Create Lambda Layer with pymysql',
                       '3. Configure security groups properly',
                       '4. Use AWS Secrets Manager for credentials'
                   ]
               })
           }
       
       return {
           'statusCode': 405,
           'body': json.dumps({'error': 'Method not allowed'})
       }
   ```

5. **Add Environment Variables**
   - Go to Configuration → Environment variables
   - Click "Edit" → "Add environment variable"
   - Add these variables:
     - `DB_HOST`: your-rds-endpoint
     - `DB_USER`: admin
     - `DB_NAME`: cloudlab
   - **Important**: Never store passwords in code or environment variables in production! Use AWS Secrets Manager instead.

6. **Test the Lambda Function**
   - Create a test event with:
   ```json
   {
     "httpMethod": "GET"
   }
   ```
   - Run the test
   - You should see a successful response with database configuration info

### Part 6: Database Maintenance and Monitoring

1. **Create Manual Snapshot**
   - Go to RDS Console
   - Select your database
   - Actions → Take snapshot
   - **Snapshot name**: `cloudlab-snapshot-[date]`
   - Click "Take snapshot"
   - Wait 2-5 minutes for completion

2. **Monitor Database Performance**
   - In RDS Console, click "Monitoring" tab
   - Observe metrics like:
     - CPU utilization
     - Database connections
     - Free storage space
     - Network throughput
   - Click on any metric to see detailed CloudWatch graphs

3. **Check Automated Backups**
   - Click "Maintenance & backups" tab
   - View backup retention settings (7 days)
   - See automated backup schedule
   - Note the backup window

4. **View Database Logs**
   - Click "Logs & events" tab
   - View recent events
   - Download error logs if needed

### Part 7: Database Security Best Practices

1. **Review Security Group Rules**
   ```bash
   # From EC2, check current connections
   mysql -h your-rds-endpoint -u admin -p -e "SHOW PROCESSLIST;"
   ```

2. **Create Read-Only Database User**
   ```sql
   -- Connect to database
   mysql -h your-rds-endpoint -u admin -p
   
   -- Create read-only user
   CREATE USER 'readonly'@'%' IDENTIFIED BY 'ReadOnly2025!';
   GRANT SELECT ON cloudlab.* TO 'readonly'@'%';
   FLUSH PRIVILEGES;
   
   -- Test the read-only user
   EXIT;
   mysql -h your-rds-endpoint -u readonly -p
   -- Enter password: ReadOnly2025!
   
   USE cloudlab;
   SELECT * FROM students;  -- This should work
   INSERT INTO students (name, email, major, year, gpa) 
   VALUES ('Test', 'test@test.com', 'CS', 1, 3.0);  -- This should fail
   
   EXIT;
   ```

3. **Security Checklist**
   ```markdown
   ✓ RDS in private subnet (for production)
   ✓ Security group limits access to specific sources
   ✓ Strong master password
   ✓ SSL/TLS encryption enabled (default in RDS)
   ✓ Automated backups enabled
   ✓ Deletion protection (should enable for production)
   ✓ Read-only users for analytics
   ✓ Never commit credentials to code
   ✓ Use Secrets Manager for production credentials
   ```

## Challenge Tasks

Try these advanced exercises:

1. **Multi-AZ Deployment**: Research how to enable Multi-AZ for high availability (don't actually enable it - costs money!)

2. **Read Replica**: Understand how to create read replicas for scaling read operations

3. **Parameter Groups**: Create a custom parameter group with modified settings

4. **Enhanced Monitoring**: Calculate the cost of enabling enhanced monitoring

## Cleanup

**IMPORTANT - To avoid charges**:

1. **STOP your Flask application** (Ctrl+C in terminal)

2. **DELETE your RDS database**:
   - Go to RDS Console
   - Select your database
   - Actions → Delete
   - **Uncheck** "Create final snapshot" (to save time and storage costs)
   - **Uncheck** "Retain automated backups"
   - Type `delete me` to confirm
   - Click "Delete"
   - **This will take 5-10 minutes**

3. **Delete manual snapshots**:
   - Go to Snapshots
   - Select your manual snapshot
   - Actions → Delete snapshot

4. **Stop EC2 instance**:
   - Go to EC2 Console
   - Select instance → Instance state → Stop

5. **Verify deletion**:
   - Check RDS Console shows no databases
   - Check Snapshots shows no snapshots
   - This is crucial to avoid charges!

## Key Takeaways

- **RDS** provides managed database services without operational overhead
- **Security groups** control network access to your database
- **Automated backups** and **snapshots** provide data protection
- **LabRole** should be used for all Lambda functions in Learner Labs
- **Multi-AZ** deployments provide high availability (for production)
- **Read replicas** can improve read performance
- **CloudWatch** provides comprehensive database monitoring
- **Never hardcode credentials** - use environment variables or Secrets Manager
- **Always delete RDS instances** when done to avoid charges

## Additional Resources

- [Amazon RDS User Guide](https://docs.aws.amazon.com/rds/latest/userguide/)
- [RDS Best Practices](https://docs.aws.amazon.com/rds/latest/userguide/CHAP_BestPractices.html)
- [RDS Pricing](https://aws.amazon.com/rds/pricing/)
- [MySQL on RDS](https://docs.aws.amazon.com/rds/latest/userguide/CHAP_MySQL.html)
- [Lambda with RDS](https://docs.aws.amazon.com/lambda/latest/dg/services-rds.html)

## Troubleshooting

**Can't connect to RDS from EC2?**
- Verify RDS status is "Available"
- Check security group allows port 3306 from your EC2's security group
- Ensure you're using the correct endpoint hostname
- Verify database credentials are correct
- Check RDS is in same region as your EC2 instance

**Web application can't connect to database?**
- Verify database credentials in app.py
- Check that PyMySQL is installed: `pip3 list | grep PyMySQL`
- Ensure RDS endpoint is correctly specified in code
- Look at Flask error messages in terminal for specific issues
- Verify EC2 security group is allowed in RDS security group

**Lambda function timing out?**
- Lambda needs to be in same VPC as RDS for private connectivity
- Database connections can be slow to establish
- Consider using RDS Proxy for connection pooling
- Increase Lambda timeout to 30 seconds (Configuration → General configuration)

**"Access denied" errors in MySQL?**
- Verify you're using correct username and password
- Check user has necessary permissions
- Ensure you're connecting to correct database
- Try resetting password in RDS console if needed

**High database costs?**
- Ensure you're using db.t3.micro or db.t2.micro (free tier)
- Delete the database when not actively using it
- Don't enable Multi-AZ in Learner Labs
- Use automated backups instead of many manual snapshots
- Monitor your usage in AWS Cost Explorer

**"Cannot create RDS instance"?**
- Check you haven't exceeded free tier limits
- Verify you have available VPC resources
- Ensure you're in a supported region
- Check Learner Labs hasn't restricted RDS creation

---

**Next Week Preview**: We'll explore VPC networking and learn how to build secure, multi-tier architectures in the cloud!
