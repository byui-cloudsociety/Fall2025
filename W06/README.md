# Week 06 - RDS & Databases Lab

This week we'll dive into Amazon RDS (Relational Database Service) and learn how to manage databases in the cloud without the operational overhead!

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
- **RDS Free Tier**: 750 hours of db.t2.micro or db.t3.micro instances
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
   - **Engine version**: MySQL 8.0.35 (latest)
   - **Templates**: Free tier
   - **DB instance identifier**: `cloud-lab-db-[your-name]`
   - **Master username**: `admin`
   - **Master password**: `CloudLab2025!` (write this down!)
   - **Confirm password**: `CloudLab2025!`

3. **Instance Configuration**
   - **DB instance class**: db.t3.micro (Free tier eligible)
   - **Storage type**: General Purpose SSD (gp2)
   - **Allocated storage**: 20 GB (minimum)
   - **Enable storage autoscaling**: Uncheck for cost control

4. **Connectivity Settings**
   - **VPC**: Default VPC
   - **Subnet group**: default
   - **Public access**: **Yes** (for easier connection in lab)
   - **VPC security group**: Create new
   - **Security group name**: `rds-mysql-sg`
   - **Availability Zone**: No preference

5. **Database Authentication**
   - **Database authentication**: Password authentication

6. **Additional Configuration**
   - **Initial database name**: `cloudlab`
   - **DB parameter group**: default.mysql8.0
   - **Backup retention period**: 7 days
   - **Enable automated backups**: Yes
   - **Backup window**: Default
   - **Enable Enhanced monitoring**: No (to save costs)
   - **Enable Performance Insights**: No (to save costs)

7. **Create Database**
   - Review settings
   - **Estimated monthly costs** should show free tier
   - Click "Create database"
   - Wait 5-10 minutes for creation

### Part 2: Configure Security Groups

1. **Find Your RDS Security Group**
   - Go to EC2 Console → Security Groups
   - Find the RDS security group created (starts with `rds-launch-wizard`)

2. **Edit Inbound Rules**
   - Select the RDS security group
   - Click "Edit inbound rules"
   - Add rule:
     - **Type**: MySQL/Aurora
     - **Port**: 3306
     - **Source**: My IP (your current IP)
   - Click "Save rules"

3. **Note the Database Endpoint**
   - Go back to RDS Console
   - Click on your database instance
   - Copy the **Endpoint** (looks like: `cloud-lab-db-yourname.xxxxxxxxx.us-east-1.rds.amazonaws.com`)

### Part 3: Connect from EC2

1. **Start Your EC2 Instance**
   - Go to EC2 Console
   - Start your instance from previous weeks
   - Connect via SSH

2. **Install MySQL Client**
   ```bash
   # Update system
   sudo yum update -y
   
   # Install MySQL client
   sudo yum install -y mysql
   
   # Verify installation
   mysql --version
   ```

3. **Connect to RDS Database**
   ```bash
   # Connect to your RDS instance (replace with your endpoint)
   mysql -h cloud-lab-db-yourname.xxxxxxxxx.us-east-1.rds.amazonaws.com -u admin -p
   # Enter password: CloudLab2025!
   ```

4. **Basic Database Operations**
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
   
   -- Update a student's information
   UPDATE students SET gpa = 3.80 WHERE name = 'John Smith';
   
   -- Verify update
   SELECT * FROM students WHERE name = 'John Smith';
   ```

### Part 4: Create a Web Application with Database

1. **Install Required Packages on EC2**
   ```bash
   # Install Python and pip if not already installed
   sudo yum install -y python3 python3-pip
   
   # Install required Python packages
   pip3 install flask pymysql
   ```

2. **Create a Simple Web Application**
   ```bash
   # Create app directory
   mkdir ~/student-app
   cd ~/student-app
   
   # Create the Flask application
   cat > app.py << 'EOF'
   from flask import Flask, render_template, request, redirect, url_for, flash
   import pymysql
   import os
   
   app = Flask(__name__)
   app.secret_key = 'your-secret-key-here'
   
   # Database configuration
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
           flash(f'Database error: {str(e)}')
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
               
               flash('Student added successfully!')
               return redirect(url_for('index'))
           except Exception as e:
               flash(f'Error adding student: {str(e)}')
       
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
               
               cursor.execute("SELECT major, COUNT(*) as count FROM students GROUP BY major")
               major_stats = cursor.fetchall()
               
           conn.close()
           
           return render_template('stats.html', 
                                total=total, 
                                avg_gpa=round(avg_gpa, 2) if avg_gpa else 0,
                                major_stats=major_stats)
       except Exception as e:
           flash(f'Database error: {str(e)}')
           return render_template('stats.html', total=0, avg_gpa=0, major_stats=[])
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000, debug=True)
   EOF
   
   # Replace the RDS endpoint in the file
   # Edit the file and replace YOUR_RDS_ENDPOINT_HERE with your actual endpoint
   nano app.py
   ```

3. **Create HTML Templates**
   ```bash
   # Create templates directory
   mkdir templates
   
   # Create base template
   cat > templates/base.html << 'EOF'
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>{% block title %}Student Management{% endblock %}</title>
       <style>
           body { font-family: Arial, sans-serif; margin: 40px; background: #f5f5f5; }
           .container { max-width: 800px; margin: 0 auto; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
           .nav { margin-bottom: 20px; }
           .nav a { margin-right: 15px; text-decoration: none; color: #007bff; font-weight: bold; }
           .nav a:hover { text-decoration: underline; }
           .flash { padding: 10px; margin: 10px 0; border-radius: 5px; }
           .flash.success { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
           .flash.error { background: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
           table { width: 100%; border-collapse: collapse; margin-top: 20px; }
           th, td { padding: 12px; text-align: left; border-bottom: 1px solid #ddd; }
           th { background-color: #f8f9fa; font-weight: bold; }
           .form-group { margin-bottom: 15px; }
           label { display: block; margin-bottom: 5px; font-weight: bold; }
           input, select { width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 4px; box-sizing: border-box; }
           button { background: #007bff; color: white; padding: 10px 20px; border: none; border-radius: 4px; cursor: pointer; }
           button:hover { background: #0056b3; }
       </style>
   </head>
   <body>
       <div class="container">
           <div class="nav">
               <a href="/">All Students</a>
               <a href="/add">Add Student</a>
               <a href="/stats">Statistics</a>
           </div>
           
           {% with messages = get_flashed_messages() %}
               {% if messages %}
                   {% for message in messages %}
                       <div class="flash">{{ message }}</div>
                   {% endfor %}
               {% endif %}
           {% endwith %}
           
           {% block content %}{% endblock %}
       </div>
   </body>
   </html>
   EOF
   
   # Create index template
   cat > templates/index.html << 'EOF'
   {% extends "base.html" %}
   
   {% block title %}Student List{% endblock %}
   
   {% block content %}
   <h1>Student Management System</h1>
   <p>Connected to RDS MySQL Database</p>
   
   {% if students %}
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
               <td>{{ student.created_at.strftime('%Y-%m-%d') }}</td>
           </tr>
           {% endfor %}
       </tbody>
   </table>
   {% else %}
   <p>No students found. <a href="/add">Add the first student!</a></p>
   {% endif %}
   {% endblock %}
   EOF
   
   # Create add student template
   cat > templates/add_student.html << 'EOF'
   {% extends "base.html" %}
   
   {% block title %}Add Student{% endblock %}
   
   {% block content %}
   <h1>Add New Student</h1>
   
   <form method="POST">
       <div class="form-group">
           <label for="name">Name:</label>
           <input type="text" id="name" name="name" required>
       </div>
       
       <div class="form-group">
           <label for="email">Email:</label>
           <input type="email" id="email" name="email" required>
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
           </select>
       </div>
       
       <div class="form-group">
           <label for="year">Year:</label>
           <select id="year" name="year" required>
               <option value="">Select Year</option>
               <option value="1">Freshman</option>
               <option value="2">Sophomore</option>
               <option value="3">Junior</option>
               <option value="4">Senior</option>
           </select>
       </div>
       
       <div class="form-group">
           <label for="gpa">GPA:</label>
           <input type="number" id="gpa" name="gpa" min="0" max="4" step="0.01" required>
       </div>
       
       <button type="submit">Add Student</button>
   </form>
   {% endblock %}
   EOF
   
   # Create stats template
   cat > templates/stats.html << 'EOF'
   {% extends "base.html" %}
   
   {% block title %}Statistics{% endblock %}
   
   {% block content %}
   <h1>Student Statistics</h1>
   
   <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin: 20px 0;">
       <div style="background: #e9ecef; padding: 20px; border-radius: 8px; text-align: center;">
           <h3>Total Students</h3>
           <p style="font-size: 2em; color: #007bff; margin: 0;">{{ total }}</p>
       </div>
       
       <div style="background: #e9ecef; padding: 20px; border-radius: 8px; text-align: center;">
           <h3>Average GPA</h3>
           <p style="font-size: 2em; color: #28a745; margin: 0;">{{ avg_gpa }}</p>
       </div>
   </div>
   
   <h3>Students by Major</h3>
   {% if major_stats %}
   <table>
       <thead>
           <tr>
               <th>Major</th>
               <th>Number of Students</th>
           </tr>
       </thead>
       <tbody>
           {% for stat in major_stats %}
           <tr>
               <td>{{ stat.major }}</td>
               <td>{{ stat.count }}</td>
           </tr>
           {% endfor %}
       </tbody>
   </table>
   {% else %}
   <p>No data available.</p>
   {% endif %}
   {% endblock %}
   EOF
   ```

4. **Run the Web Application**
   ```bash
   # Make sure you're in the student-app directory
   cd ~/student-app
   
   # Run the Flask application
   python3 app.py
   ```

5. **Test Your Web Application**
   - Open a new terminal/browser
   - Visit: `http://[your-ec2-public-ip]:5000`
   - Add some students and test the functionality

### Part 5: Connect Lambda to RDS

1. **Create Database Lambda Function**
   - Go to Lambda Console
   - Create function: `student-api-[your-name]`
   - Runtime: Python 3.11

2. **Add RDS Connection Code**
   ```python
   import json
   import pymysql
   import os
   
   # Database configuration
   DB_HOST = 'YOUR_RDS_ENDPOINT_HERE'  # Replace with your endpoint
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
   
   def lambda_handler(event, context):
       try:
           http_method = event.get('httpMethod', 'GET')
           
           if http_method == 'GET':
               # Get all students
               conn = get_db_connection()
               with conn.cursor() as cursor:
                   cursor.execute("SELECT * FROM students ORDER BY name")
                   students = cursor.fetchall()
               conn.close()
               
               # Convert datetime objects to strings for JSON serialization
               for student in students:
                   if 'created_at' in student:
                       student['created_at'] = student['created_at'].isoformat()
               
               return {
                   'statusCode': 200,
                   'headers': {
                       'Content-Type': 'application/json',
                       'Access-Control-Allow-Origin': '*'
                   },
                   'body': json.dumps({
                       'students': students,
                       'count': len(students)
                   })
               }
           
           elif http_method == 'POST':
               # Add new student
               body = json.loads(event.get('body', '{}'))
               
               required_fields = ['name', 'email', 'major', 'year', 'gpa']
               for field in required_fields:
                   if field not in body:
                       return {
                           'statusCode': 400,
                           'body': json.dumps({'error': f'Missing required field: {field}'})
                       }
               
               conn = get_db_connection()
               with conn.cursor() as cursor:
                   cursor.execute(
                       "INSERT INTO students (name, email, major, year, gpa) VALUES (%s, %s, %s, %s, %s)",
                       (body['name'], body['email'], body['major'], body['year'], body['gpa'])
                   )
               conn.commit()
               conn.close()
               
               return {
                   'statusCode': 201,
                   'headers': {
                       'Content-Type': 'application/json',
                       'Access-Control-Allow-Origin': '*'
                   },
                   'body': json.dumps({'message': 'Student added successfully'})
               }
           
           else:
               return {
                   'statusCode': 405,
                   'body': json.dumps({'error': 'Method not allowed'})
               }
               
       except Exception as e:
           return {
               'statusCode': 500,
               'body': json.dumps({'error': str(e)})
           }
   ```

3. **Add PyMySQL Layer**
   - The Lambda function needs the PyMySQL library
   - Go to Configuration → Layers
   - Add a layer (you'll need to create a deployment package or use a public layer)
   - For this lab, we'll simulate the functionality

### Part 6: Database Maintenance and Monitoring

1. **Create Manual Snapshot**
   - Go to RDS Console
   - Select your database
   - Actions → Take snapshot
   - Name: `cloudlab-manual-snapshot-[date]`

2. **Monitor Database Performance**
   - In RDS Console, click "Monitoring" tab
   - Observe metrics like:
     - CPU utilization
     - Database connections
     - Read/Write IOPS
     - Free storage space

3. **Check Automated Backups**
   - Click "Maintenance & backups" tab
   - View backup retention settings
   - See automated backup schedule

4. **Parameter Groups (Optional)**
   - Go to Parameter groups in RDS sidebar
   - Create custom parameter group
   - Modify database settings like:
     - `max_connections`
     - `innodb_buffer_pool_size`

## Challenge
- Configure Multi-AZ deployment for high availability

## Cleanup

**To avoid charges**:
1. **STOP your EC2 instance** (Ctrl+C to stop Flask app first)
2. **DELETE your RDS database**:
   - Go to RDS Console
   - Select your database
   - Actions → Delete
   - **Uncheck** "Create final snapshot" (saves time and money)
   - Type "delete me" to confirm
   - This will take several minutes

3. **Delete snapshots** if you created any for testing
4. **Keep Lambda functions** (no cost when not running)

## Key Takeaways

- **RDS** provides managed database services without operational overhead
- **Security groups** control network access to your database
- **Automated backups** and **snapshots** provide data protection
- **Multi-AZ** deployments provide high availability
- **Read replicas** can improve read performance
- **Parameter groups** allow database configuration customization
- **CloudWatch** provides comprehensive database monitoring

## Additional Resources

- [Amazon RDS User Guide](https://docs.aws.amazon.com/rds/latest/userguide/)
- [RDS Best Practices](https://docs.aws.amazon.com/rds/latest/userguide/CHAP_BestPractices.html)
- [RDS Pricing](https://aws.amazon.com/rds/pricing/)
- [MySQL on RDS](https://docs.aws.amazon.com/rds/latest/userguide/CHAP_MySQL.html)

## Troubleshooting

**Can't connect to RDS from EC2?**
- Check security group allows port 3306 from EC2's security group
- Verify RDS is in "Available" state
- Ensure you're using the correct endpoint and credentials

**Web application can't connect to database?**
- Verify database credentials in app.py
- Check that PyMySQL is installed: `pip3 list | grep PyMySQL`
- Look at Flask error messages for specific connection issues

**Lambda function timing out?**
- Database connections can be slow to establish
- Consider using connection pooling
- Increase Lambda timeout to 30 seconds

**High database costs?**
- Ensure you're using db.t3.micro (free tier)
- Delete the database when not in use
- Use automated backups instead of many manual snapshots

---

