# Week 05 - Lambda & Serverless Lab

Welcome to the world of serverless computing! This week we'll explore AWS Lambda and learn how to run code without managing servers.

## Learning Objectives

By the end of this lab, you will be able to:

- Create and deploy Lambda functions
- Understand serverless computing concepts
- Trigger Lambda functions with various event sources
- Connect Lambda with S3 and other AWS services
- Build a serverless image processor
- Monitor Lambda function performance
- Understand Lambda pricing and optimization

## Cost Awareness

- **Lambda Free Tier**: 1M free requests + 400,000 GB-seconds compute time/month
- **Typical cost**: $0.20 per 1M requests + $0.0000166667 per GB-second
- We'll use less than $0.50 of your budget today
- **No servers to manage** = no idle time costs!

## Lab Instructions

### Part 1: Your First Lambda Function

1. **Navigate to Lambda**
   - In AWS Console, search for "Lambda"
   - Click "Create function"

2. **Create Hello World Function**
   - Choose "Author from scratch"
   - **Function name**: `hello-world-[your-name]`
   - **Runtime**: Python 3.11
   - **Architecture**: x86_64
   - Leave other settings default
   - Click "Create function"

3. **Explore the Lambda Console**
   - Notice the code editor with default code
   - See the "Test" button and configuration tabs
   - Explore the "Monitor" tab (CloudWatch integration)

4. **Test Your Function**
   - Click "Test" button
   - Create new test event:
     - **Event name**: `test-event`
     - Keep default "hello-world" template
     - Click "Save"
   - Click "Test" again
   - View execution results and logs

### Part 2: Build a Simple Calculator Function

1. **Create Calculator Function**
   - Create new function: `calculator-[your-name]`
   - Runtime: Python 3.11

2. **Replace Default Code**

   ```python
   import json
   import math

   def lambda_handler(event, context):
       try:
           # Extract operation and numbers from event
           operation = event.get('operation', 'add')
           num1 = float(event.get('num1', 0))
           num2 = float(event.get('num2', 0))
           
           # Perform calculation
           if operation == 'add':
               result = num1 + num2
           elif operation == 'subtract':
               result = num1 - num2
           elif operation == 'multiply':
               result = num1 * num2
           elif operation == 'divide':
               if num2 == 0:
                   raise ValueError("Cannot divide by zero")
               result = num1 / num2
           elif operation == 'power':
               result = num1 ** num2
           elif operation == 'sqrt':
               if num1 < 0:
                   raise ValueError("Cannot take square root of negative number")
               result = math.sqrt(num1)
           else:
               raise ValueError(f"Unsupported operation: {operation}")
           
           # Return successful response
           return {
               'statusCode': 200,
               'body': json.dumps({
                   'operation': operation,
                   'num1': num1,
                   'num2': num2 if operation != 'sqrt' else None,
                   'result': result,
                   'message': f'{num1} {operation} {num2 if operation != "sqrt" else ""} = {result}'
               })
           }
           
       except Exception as e:
           # Return error response
           return {
               'statusCode': 400,
               'body': json.dumps({
                   'error': str(e),
                   'message': 'Calculation failed'
               })
           }
   ```

3. **Deploy and Test**
   - Click "Deploy"
   - Create test events for different operations:

   **Addition Test:**

   ```json
   {
     "operation": "add",
     "num1": 10,
     "num2": 5
   }
   ```

   **Division Test:**

   ```json
   {
     "operation": "divide",
     "num1": 100,
     "num2": 20
   }
   ```

   **Square Root Test:**

   ```json
   {
     "operation": "sqrt",
     "num1": 64
   }
   ```

### Part 3: S3-Triggered Lambda Function

1. **Create Image Processor Function**
   - Function name: `image-processor-[your-name]`
   - Runtime: Python 3.11
   - **Timeout**: 30 seconds (Configuration → General configuration)

2. **Add the Code**

   ```python
   import json
   import boto3
   import urllib.parse
   from datetime import datetime

   # Initialize S3 client
   s3_client = boto3.client('s3')

   def lambda_handler(event, context):
       try:
           # Parse S3 event
           for record in event['Records']:
               # Get bucket and object key from event
               bucket = record['s3']['bucket']['name']
               key = urllib.parse.unquote_plus(record['s3']['object']['key'])
               
               # Get object information
               response = s3_client.head_object(Bucket=bucket, Key=key)
               file_size = response['ContentLength']
               last_modified = response['LastModified']
               content_type = response.get('ContentType', 'Unknown')
               
               # Create log entry
               log_entry = {
                   'timestamp': datetime.utcnow().isoformat(),
                   'bucket': bucket,
                   'object_key': key,
                   'file_size': file_size,
                   'content_type': content_type,
                   'last_modified': last_modified.isoformat(),
                   'processing_status': 'completed'
               }
               
               # Log the processing (in real app, might save to database)
               print(f"Processed file: {json.dumps(log_entry, indent=2)}")
               
               # Simple file analysis
               if content_type.startswith('image/'):
                   print(f"✅ Image file detected: {key}")
                   # Here you could add image processing logic
                   # Like resizing, format conversion, etc.
               elif content_type.startswith('text/'):
                   print(f"📄 Text file detected: {key}")
                   # Here you could add text processing logic
               else:
                   print(f"📁 Other file type: {content_type}")
           
           return {
               'statusCode': 200,
               'body': json.dumps({
                   'message': f'Successfully processed {len(event["Records"])} file(s)',
                   'processed_files': [record['s3']['object']['key'] for record in event['Records']]
               })
           }
           
       except Exception as e:
           print(f"Error processing S3 event: {str(e)}")
           return {
               'statusCode': 500,
               'body': json.dumps({
                   'error': str(e),
                   'message': 'Failed to process S3 event'
               })
           }
   ```

3. **Add S3 Permissions**
   - Go to Configuration → Permissions
   - Click on the execution role (it will open IAM in a new tab)
   - Click "Add permissions" → "Attach policies"
   - Search for and attach: `AmazonS3ReadOnlyAccess`

4. **Connect S3 Bucket to Lambda**
   - Go to your S3 bucket from Week 04
   - Click "Properties" tab
   - Scroll to "Event notifications"
   - Click "Create event notification":
     - **Name**: `lambda-trigger`
     - **Event types**: Check "All object create events"
     - **Destination**: Lambda function
     - **Lambda function**: Select your `image-processor` function
   - Save

5. **Test the Trigger**
   - Upload a new file to your S3 bucket (any file type)
   - Go back to Lambda console
   - Check "Monitor" tab → "View CloudWatch logs"
   - You should see your function executed automatically!

### Part 4: Build a URL Shortener API

1. **Create URL Shortener Function**
   - Function name: `url-shortener-[your-name]`
   - Runtime: Python 3.11

2. **Add the Code**

   ```python
   import json
   import string
   import random
   import boto3
   from datetime import datetime, timedelta

   # Simple in-memory storage (in production, use DynamoDB)
   url_database = {}

   def generate_short_code(length=6):
       """Generate a random short code"""
       chars = string.ascii_letters + string.digits
       return ''.join(random.choice(chars) for _ in range(length))

   def lambda_handler(event, context):
       try:
           # Parse the request
           http_method = event.get('httpMethod', 'GET')
           path = event.get('path', '/')
           body = event.get('body', '{}')
           
           if http_method == 'POST':
               # Create short URL
               data = json.loads(body) if body else {}
               original_url = data.get('url')
               
               if not original_url:
                   return {
                       'statusCode': 400,
                       'headers': {'Content-Type': 'application/json'},
                       'body': json.dumps({'error': 'URL is required'})
                   }
               
               # Generate short code
               short_code = generate_short_code()
               while short_code in url_database:  # Ensure uniqueness
                   short_code = generate_short_code()
               
               # Store in database
               url_database[short_code] = {
                   'original_url': original_url,
                   'created_at': datetime.utcnow().isoformat(),
                   'clicks': 0
               }
               
               return {
                   'statusCode': 201,
                   'headers': {'Content-Type': 'application/json'},
                   'body': json.dumps({
                       'short_code': short_code,
                       'short_url': f'https://your-api-gateway-url/{short_code}',
                       'original_url': original_url
                   })
               }
           
           elif http_method == 'GET':
               # Redirect or get stats
               short_code = path.strip('/')
               
               if short_code == '' or short_code == 'stats':
                   # Return statistics
                   stats = {
                       'total_urls': len(url_database),
                       'total_clicks': sum(data['clicks'] for data in url_database.values()),
                       'urls': [
                           {
                               'short_code': code,
                               'original_url': data['original_url'],
                               'clicks': data['clicks'],
                               'created_at': data['created_at']
                           }
                           for code, data in url_database.items()
                       ]
                   }
                   
                   return {
                       'statusCode': 200,
                       'headers': {'Content-Type': 'application/json'},
                       'body': json.dumps(stats)
                   }
               
               # Try to redirect
               if short_code in url_database:
                   url_database[short_code]['clicks'] += 1
                   original_url = url_database[short_code]['original_url']
                   
                   return {
                       'statusCode': 302,
                       'headers': {
                           'Location': original_url,
                           'Content-Type': 'application/json'
                       },
                       'body': json.dumps({'redirecting_to': original_url})
                   }
               else:
                   return {
                       'statusCode': 404,
                       'headers': {'Content-Type': 'application/json'},
                       'body': json.dumps({'error': 'Short URL not found'})
                   }
           
           else:
               return {
                   'statusCode': 405,
                   'headers': {'Content-Type': 'application/json'},
                   'body': json.dumps({'error': 'Method not allowed'})
               }
               
       except Exception as e:
           return {
               'statusCode': 500,
               'headers': {'Content-Type': 'application/json'},
               'body': json.dumps({'error': str(e)})
           }
   ```

3. **Test the URL Shortener**
   - Create test events for different scenarios:

   **Create Short URL:**

   ```json
   {
     "httpMethod": "POST",
     "path": "/",
     "body": "{\"url\": \"https://aws.amazon.com/lambda/\"}"
   }
   ```

   **Get Stats:**

   ```json
   {
     "httpMethod": "GET",
     "path": "/stats"
   }
   ```

### Part 5: Lambda Performance and Monitoring

1. **Explore CloudWatch Metrics**
   - Go to any of your Lambda functions
   - Click "Monitor" tab
   - Observe metrics like:
     - Invocations
     - Duration
     - Error count
     - Success rate

2. **View Detailed Logs**
   - Click "View CloudWatch logs"
   - Explore log groups and streams
   - Notice how each execution creates log entries

3. **Performance Testing**
   - Run your calculator function multiple times with different inputs
   - Notice "cold start" vs "warm start" execution times
   - Try increasing memory allocation (Configuration → General configuration)

### Part 6: Connect Lambda with Your Static Website

1. **Update Your S3 Website**
   - Add this HTML to your `index.html` (in a new section):

   ```html
   <section id="calculator">
       <h2>Serverless Calculator</h2>
       <p>This calculator runs on AWS Lambda!</p>
       
       <div class="calc-form">
           <input type="number" id="num1" placeholder="First number" value="10">
           <select id="operation">
               <option value="add">Add (+)</option>
               <option value="subtract">Subtract (-)</option>
               <option value="multiply">Multiply (×)</option>
               <option value="divide">Divide (÷)</option>
               <option value="power">Power (^)</option>
               <option value="sqrt">Square Root (√)</option>
           </select>
           <input type="number" id="num2" placeholder="Second number" value="5">
           <button onclick="calculate()">Calculate</button>
           
           <div id="result"></div>
       </div>
       
       <script>
       function calculate() {
           const num1 = document.getElementById('num1').value;
           const num2 = document.getElementById('num2').value;
           const operation = document.getElementById('operation').value;
           
           // In a real application, you'd call your Lambda function via API Gateway
           // For now, we'll simulate the calculation
           let result;
           const n1 = parseFloat(num1);
           const n2 = parseFloat(num2);
           
           switch(operation) {
               case 'add': result = n1 + n2; break;
               case 'subtract': result = n1 - n2; break;
               case 'multiply': result = n1 * n2; break;
               case 'divide': result = n2 !== 0 ? n1 / n2 : 'Error: Division by zero'; break;
               case 'power': result = Math.pow(n1, n2); break;
               case 'sqrt': result = n1 >= 0 ? Math.sqrt(n1) : 'Error: Negative number'; break;
           }
           
           document.getElementById('result').innerHTML = 
               `<h3>Result: ${result}</h3>
                <p><em>This calculation was simulated in the browser. In Week 6, we'll learn to connect this to our actual Lambda function via API Gateway!</em></p>`;
       }
       </script>
   </section>
   ```

2. **Add CSS for the Calculator**
   Add this to your `style.css`:

   ```css
   .calc-form {
       background: #f8f9fa;
       padding: 1.5rem;
       border-radius: 10px;
       margin-top: 1rem;
   }
   
   .calc-form input, .calc-form select, .calc-form button {
       margin: 0.5rem;
       padding: 0.75rem;
       border: 2px solid #ddd;
       border-radius: 5px;
       font-size: 1rem;
   }
   
   .calc-form button {
       background: #667eea;
       color: white;
       border-color: #667eea;
       cursor: pointer;
       transition: all 0.3s;
   }
   
   .calc-form button:hover {
       background: #5a6fd8;
       transform: translateY(-1px);
   }
   
   #result {
       margin-top: 1rem;
       padding: 1rem;
       background: #e8f5e8;
       border-radius: 5px;
       border-left: 4px solid #28a745;
   }
   ```

3. **Upload Updated Files**
   - Upload the updated HTML and CSS files to your S3 bucket
   - Visit your website to see the new calculator section

## Challenge

- Build a serverless contact form that sends emails via SES

## Cleanup

**To manage costs**:

1. **Keep your Lambda functions** (they're free when not running)
2. **Delete any large test files** from S3
3. **Monitor your usage** in the billing dashboard
4. **Stop EC2 instance** if still running

## Key Takeaways

- **Serverless** = no server management, pay only for execution time
- **Event-driven** architecture enables automatic responses to changes
- **Lambda functions** can be triggered by many AWS services
- **Cold starts** affect initial execution time but subsequent calls are faster
- **CloudWatch** provides comprehensive monitoring and logging
- **Serverless** is perfect for microservices and event processing

## Additional Resources

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [Serverless Application Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/)
- [Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
- [Python on Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html)

## Troubleshooting

**Lambda function timing out?**

- Increase timeout in Configuration → General configuration
- Check for infinite loops or blocking operations
- Optimize code for faster execution

**S3 trigger not working?**

- Verify Lambda has S3 read permissions
- Check S3 event notification configuration
- Look for error messages in CloudWatch logs

**Function not executing?**

- Check IAM permissions for the Lambda execution role
- Verify trigger configuration
- Look for syntax errors in code

**Import errors in Lambda?**

- Only standard Python libraries are available by default
- For additional packages, you need to create deployment packages
- boto3 and botocore are pre-installed

---

**Next Week Preview**: We'll explore RDS and learn how to set up managed databases in the cloud!
