# Week 09

# ☁️ Cloud-a-Thon Project Ideas (Beginner-Friendly)

Welcome to the Cloud-a-Thon!  
You’ll have **about 2 hours** to build a small cloud project using AWS services you’ve learned so far (EC2, Lambda, S3, RDS, DynamoDB, etc).  
You can work **solo or in teams**. Pick one of these ideas or come up with your own!

**AI use is encouraged!**

---

## 1️⃣ Serverless To-Do App

**Goal:** Create a simple To-Do list that saves tasks in the cloud.  
**Services:** DynamoDB, AWS Lambda, API Gateway (or Lambda Function URLs), S3 or EC2 (for frontend).  
**Description:**

- Build Lambda functions for Create, Read, Update, Delete (CRUD).
- Store tasks in a DynamoDB table.
- Serve a small HTML/JS page from S3 or EC2 that interacts with your API.  
  **Stretch Idea:** Add task filtering or completion status.

---

## 2️⃣ Image Uploader + Thumbnail Generator

**Goal:** Upload an image to S3 and automatically generate a smaller version.  
**Services:** S3, Lambda (S3 trigger).  
**Description:**

- Upload images via a signed URL or S3 console.
- When a new image is uploaded, trigger a Lambda to create a thumbnail or log details.  
  **Stretch Idea:** Display both original and thumbnail images in a simple webpage.

---

## 3️⃣ URL Shortener

**Goal:** Build a mini version of bit.ly using serverless tools.  
**Services:** DynamoDB, Lambda, API Gateway.  
**Description:**

- Store mappings of short URLs → long URLs in DynamoDB.
- Lambda generates short codes and redirects users.  
  **Stretch Idea:** Add a hit counter or expiration date.

---

## 4️⃣ Guestbook / Event Sign-In

**Goal:** Collect guest names and messages online.  
**Services:** DynamoDB **or** RDS, EC2 (Node/Express) or Lambda.  
**Description:**

- Simple form to submit a name and message.
- Display all submissions in a list.  
  **Stretch Idea:** Add timestamps or limit number of entries displayed.

---

## 5️⃣ Serverless Poll / Voting App

**Goal:** Let users vote on a topic and display real-time results.  
**Services:** DynamoDB, Lambda, S3 (frontend).  
**Description:**

- Create a poll with options stored in DynamoDB.
- Lambda updates and reads counts.
- S3 static site shows voting buttons and live results.  
  **Stretch Idea:** Allow multiple polls or user-created questions.

---

## 6️⃣ Weather Dashboard

**Goal:** Show live weather data for a chosen city.  
**Services:** Lambda (to call public weather API), S3 (frontend), DynamoDB (optional cache).  
**Description:**

- Lambda fetches weather info from a free API.
- Display data in an HTML/JS frontend.  
  **Stretch Idea:** Cache recent results in DynamoDB to reduce API calls.

---

## 7️⃣ Chat Wall / Message Board

**Goal:** A simple message board where users post short messages.  
**Services:** DynamoDB, Lambda (API), S3 or EC2 (frontend).  
**Description:**

- “Post message” and “View messages” endpoints.
- Store messages with timestamps in DynamoDB.  
  **Stretch Idea:** Add usernames or message deletion.

---

## 8️⃣ Secrets Manager Demo

**Goal:** Learn about secure data storage in the cloud.  
**Services:** S3, Lambda, IAM (roles/policies).  
**Description:**

- Simulate saving and retrieving “secrets” (like passwords or keys).
- Lambda encrypts or masks the secret when displaying it.  
  **Stretch Idea:** Use KMS encryption if available in your learner lab.

---

## 9️⃣ Static Site Deployment Pipeline

**Goal:** Deploy a small static website automatically to S3.  
**Services:** S3, (optional) GitHub Actions or CodePipeline.  
**Description:**

- Upload a static website to S3 and enable static hosting.
- Optionally set up a pipeline to redeploy when files change.  
  **Stretch Idea:** Add a version number or “last updated” display.

---

## 🔟 Page Visit Counter / Analytics

**Goal:** Count how many times a webpage has been visited.  
**Services:** DynamoDB, Lambda, S3 (frontend).  
**Description:**

- Each time the page loads, frontend calls a Lambda that increments a counter in DynamoDB.
- Display the count on the page.  
  **Stretch Idea:** Track visits by IP or browser type.

---

# Projects Recommendations by Industry Professionals

[Beginner Cloud Projects](https://www.linkedin.com/posts/lucywang-_aws-cloud-cloudcomputing-activity-7360316502715047936-pfjq?utm_source=share&utm_medium=member_desktop&rcm=ACoAABlFjNUBZrSnHLi-49lpCFPzSCAGevcBdbs)

[Cloud + Cyber Security Projects](https://www.linkedin.com/posts/lucywang-_aws-cloudsecurity-cybersecurity-activity-7380610285323259904-T_Pq?utm_source=share&utm_medium=member_desktop&rcm=ACoAABlFjNUBZrSnHLi-49lpCFPzSCAGevcBdbs)

[Intermediate Cloud Projects](https://www.linkedin.com/posts/lucywang-_top-5-intermediate-aws-cloud-projects-activity-7368651491890479105-zAVD?utm_source=share&utm_medium=member_desktop&rcm=ACoAABlFjNUBZrSnHLi-49lpCFPzSCAGevcBdbs)
