# LearnSign- Requirements Document

## 1. Overview

### 1.1 Purpose
This document outlines the requirements for LearnSign - an AI-powered platform designed to teach **Indian Sign Language (ISL)** to children ages 1-15+. Built by students for a hackathon using **5 core AWS services**.

---

## 2. Executive Summary

```
+------------------------------------------------------------+
|                    LearnSign Platform                      |
|            AI-Powered ISL Learning for Kids                |
+------------------------------------------------------------+

The platform combines:
  * Interactive video lessons
  * Gamified learning experiences  
  * Real-time sign language recognition
  * Detailed progress tracking
  * AI-powered tutoring assistance
```

---

## 3. Project Goals

### 3.1 Target Audience

| User Type | Description | Age Range |
|-----------|-------------|-----------|
| **Primary Users** | Children learning ISL | 1-15 years |
| **Secondary Users** | Parents and teachers | Adults |
| **Geographic Focus** | India | All regions |

### 3.2 Key Objectives

```
+-------------------------------------------------------------+
|                     Project Objectives                       |
+-------------------------------------------------------------+
|  1. Make ISL learning accessible and fun for children       |
|  2. Provide real-time feedback on sign accuracy             |
|  3. Track learning progress with gamification               |
|  4. Support both online and offline learning modes          |
|  5. Create a scalable, serverless architecture              |
+-------------------------------------------------------------+
```

---

## 4. Core Features

### 4.1 User Authentication

```
Feature: User Authentication System
-------------------------------------
What: Simple login/signup for students
How: Basic JWT authentication with Lambda

User Flow:
+----------+    +----------+    +----------+
|  User    |--->|  Login   |--->| Dashboard|
|  Opens   |    |  Page    |    |  Access  |
|  App     |    |          |    |          |
+----------+    +----------+    +----------+
```

**Requirements:**
- Email/password authentication
- JWT token generation
- Session management
- Password hashing with bcrypt

### 4.2 ISL Course Viewer

```
Feature: ISL Course Viewer
-------------------------------------
What: Browse and watch ISL video lessons
How: Video library stored in S3

Course Structure:
+-----------------------------------------+
|              Course Library              |
+-----------------------------------------+
|  +-- Basics                             |
|  |   +-- Greetings (Namaste, Hello)     |
|  |   +-- Common Words                   |
|  |   +-- Numbers 1-10                   |
|  +-- Alphabet                           |
|  |   +-- Letters A-M                    |
|  |   +-- Letters N-Z                    |
|  +-- Advanced                           |
|  |   +-- Sentences                      |
|  |   +-- Conversations                  |
|  +-- Practice                           |
|      +-- Interactive Quizzes            |
+-----------------------------------------+
```

**Features:**
- List of courses (Basics, Numbers, Alphabet, Common Words)
- Video playback with controls
- Simple progress tracking
- Bookmarking capability

### 4.3 ISL Sign Recognition

```
Feature: Real-time ISL Sign Recognition
-----------------------------------------
What: Real-time ISL sign recognition using webcam
How: ML model deployed on Lambda or SageMaker

Recognition Pipeline:
+------------+   +------------+   +------------+   +------------+
|  Webcam    |-->|  MediaPipe |-->|  ML Model  |-->|  Result    |
|  Capture   |   |  Landmarks |   |  Inference |   |  Display   |
+------------+   +------------+   +------------+   +------------+
```

**Features:**
- Detect hand signs via webcam
- Recognize ISL alphabet (A-Z)
- Show confidence score
- Practice mode with feedback

### 4.4 ISL Translator

```
Feature: ISL Translator
-------------------------------------
What: Type a word, see ISL sign video
How: Search database, fetch video from S3

Translation Flow:
+----------+   +----------+   +----------+
|  User    |-->|  Search  |-->|  Video   |
|  Input   |   |  Query   |   |  Display |
+----------+   +----------+   +----------+
```

**Features:**
- Search bar for word input
- Video playback of sign
- Related signs suggestions
- Favorite signs list

### 4.5 Progress Dashboard

```
Feature: Progress Dashboard
-------------------------------------
What: Track learning progress
How: Store user activity in DynamoDB

Dashboard Components:
+-------------------------------------------------------------+
|                    User Dashboard                            |
+-------------------------------------------------------------+
|  +-------------+  +-------------+  +-------------+          |
|  |  Lessons    |  |   Signs     |  |  Practice   |          |
|  |  Completed  |  |   Learned   |  |    Time     |          |
|  |    15/30    |  |    45/100   |  |   2h 30m    |          |
|  +-------------+  +-------------+  +-------------+          |
|                                                              |
|  +-----------------------------------------------+          |
|  |            Progress Chart                      |          |
|  |  ################--------  65% Complete       |          |
|  +-----------------------------------------------+          |
+-------------------------------------------------------------+
```

**Features:**
- Lessons completed counter
- Signs learned tracker
- Practice time logger
- Visual progress charts

### 4.6 AI Tutor

```
Feature: AI Tutor
-------------------------------------
What: Chat with AI about ISL
How: OpenAI API integration

AI Tutor Capabilities:
+-----------------------------------------+
|            AI Tutor Features             |
+-----------------------------------------+
|  * Answer questions about ISL           |
|  * Provide sign recommendations         |
|  * Suggest relevant video lessons       |
|  * Give learning tips and tricks        |
|  * Explain sign variations              |
+-----------------------------------------+
```

**Features:**
- Ask questions about ISL
- Get sign recommendations
- Video suggestions
- Learning path guidance

---

## 5. Technical Requirements

### 5.1 AWS Services

| Service | Purpose | Why? | Free Tier |
|---------|---------|------|-----------|
| **AWS Lambda** | Backend logic | No servers, auto-scale | 1M requests/month |
| **API Gateway** | REST API endpoints | Easy Lambda integration | 1M requests/month |
| **DynamoDB** | Database | Fast, serverless NoSQL | 25GB storage |
| **S3** | Video storage | Cheap, reliable | 5GB storage |
| **SageMaker** | ML inference | Easy model deployment | Limited |

### 5.2 Service Details

#### Service 1: AWS Lambda
```
Purpose: Run backend code (APIs, authentication, business logic)
Why: No servers to manage, pay-per-use

Lambda Functions:
+-----------------------------------------+
|           Lambda Functions               |
+-----------------------------------------+
|  * auth/login.js                        |
|  * auth/register.js                     |
|  * courses/list.js                      |
|  * courses/get.js                       |
|  * progress/get.js                      |
|  * progress/update.js                   |
|  * recognition/recognize.js             |
|  * tutor/chat.js                        |
+-----------------------------------------+
```

#### Service 2: Amazon API Gateway
```
Purpose: REST API endpoints for frontend
Why: Easy to connect Lambda functions to HTTP endpoints

API Endpoints:
+-----------------------------------------+
|            API Routes                    |
+-----------------------------------------+
|  POST /auth/login                       |
|  POST /auth/register                    |
|  GET  /courses                          |
|  GET  /courses/:courseId                |
|  GET  /progress/:userId                 |
|  POST /progress/update                  |
|  POST /recognize                        |
|  POST /tutor/chat                       |
+-----------------------------------------+
```

#### Service 3: Amazon DynamoDB
```
Purpose: NoSQL database for users, courses, progress
Why: Serverless, fast, easy to use

Tables:
+-----------------------------------------+
|           DynamoDB Tables                |
+-----------------------------------------+
|  * LearnSign-Users                      |
|    +-- userId, email, password, name    |
|  * LearnSign-Courses                    |
|    +-- courseId, title, videos          |
|  * LearnSign-Progress                   |
|    +-- userId, courseId, completed      |
+-----------------------------------------+
```

#### Service 4: Amazon S3
```
Purpose: Store ISL video files
Why: Cheap storage, easy to serve videos

S3 Buckets:
+-----------------------------------------+
|             S3 Buckets                   |
+-----------------------------------------+
|  * learnsign-videos                     |
|    +-- All ISL sign videos (.webm)      |
|  * learnsign-models                     |
|    +-- ML model files                   |
|  * learnsign-frontend                   |
|    +-- Static website files             |
+-----------------------------------------+
```

#### Service 5: Amazon SageMaker
```
Purpose: Run ISL recognition ML model
Why: Deploy TensorFlow model for real-time inference

Deployment Options:
+-----------------------------------------+
|         ML Deployment Options            |
+-----------------------------------------+
|  Option A: SageMaker Endpoint           |
|    +-- Best for production              |
|  Option B: Lambda with TensorFlow.js    |
|    +-- Best for hackathon/demo          |
+-----------------------------------------+
```

### 5.3 Frontend Requirements

```
Frontend Technology Stack:
-------------------------------------
Framework: React.js (or plain HTML/CSS/JS)
UI Library: TailwindCSS or Bootstrap
Video Player: HTML5 <video> tag
Webcam: navigator.mediaDevices.getUserMedia()
Charts: Chart.js (for dashboard)
```

**Pages Required:**
- Login/Signup page
- Course listing page
- Video player page
- Webcam capture for recognition
- Dashboard with progress
- AI Tutor chat interface

**Deployment:**
- GitHub Pages
- S3 Static Hosting
- Netlify

### 5.4 ML Model Requirements

```
ML Model Specifications:
-------------------------------------
Framework: TensorFlow/Keras LSTM model
Input: Hand landmarks from MediaPipe (21 points x 3 coordinates)
Output: Predicted ISL sign + confidence score
Training Data: 350+ ISL signs
Accuracy Target: >85%

Model Pipeline:
+----------+   +----------+   +----------+   +----------+
|  Video   |-->| MediaPipe|-->|  LSTM    |-->| Predicted|
|  Frame   |   | Landmarks|   |  Model   |   |   Sign   |
+----------+   +----------+   +----------+   +----------+
```

---

## 6. Data Models

### 6.1 Users Table (DynamoDB)

```json
{
  "TableName": "LearnSign-Users",
  "KeySchema": {
    "userId": "HASH"
  },
  "Attributes": {
    "userId": "string (UUID)",
    "email": "string",
    "password": "string (bcrypt hashed)",
    "name": "string",
    "createdAt": "number (timestamp)",
    "updatedAt": "number (timestamp)"
  },
  "GSI": "email-index"
}
```

### 6.2 Courses Table (DynamoDB)

```json
{
  "TableName": "LearnSign-Courses",
  "KeySchema": {
    "courseId": "HASH"
  },
  "Attributes": {
    "courseId": "string",
    "title": "string",
    "description": "string",
    "thumbnail": "string (S3 URL)",
    "videos": [
      {
        "id": "string",
        "title": "string",
        "url": "string (S3 URL)",
        "duration": "number (seconds)"
      }
    ]
  }
}
```

### 6.3 UserProgress Table (DynamoDB)

```json
{
  "TableName": "LearnSign-Progress",
  "KeySchema": {
    "progressId": "HASH"
  },
  "Attributes": {
    "progressId": "string (UUID)",
    "userId": "string",
    "courseId": "string",
    "completedVideos": ["videoId1", "videoId2"],
    "lastAccessed": "number (timestamp)",
    "totalTimeSpent": "number (seconds)"
  },
  "GSI": "userId-courseId-index"
}
```

---

## 7. API Endpoints

### 7.1 Authentication APIs

```
POST /auth/register
-------------------------------------
Request:
{
  "email": "user@example.com",
  "password": "securePassword123",
  "name": "Student Name"
}

Response:
{
  "success": true,
  "userId": "uuid-here",
  "message": "Registration successful"
}
```

```
POST /auth/login
-------------------------------------
Request:
{
  "email": "user@example.com",
  "password": "securePassword123"
}

Response:
{
  "success": true,
  "token": "jwt-token-here",
  "userId": "uuid-here",
  "name": "Student Name"
}
```

### 7.2 Course APIs

```
GET /courses
-------------------------------------
Response:
{
  "success": true,
  "courses": [
    {
      "courseId": "basics-101",
      "title": "ISL Basics",
      "description": "Learn fundamental ISL signs",
      "videoCount": 10
    }
  ]
}
```

```
GET /courses/:courseId
-------------------------------------
Response:
{
  "success": true,
  "course": {
    "courseId": "basics-101",
    "title": "ISL Basics",
    "videos": [
      { "id": "v1", "title": "Namaste", "url": "s3://..." }
    ]
  }
}
```

### 7.3 Progress APIs

```
GET /progress/:userId
-------------------------------------
Response:
{
  "success": true,
  "progress": {
    "totalLessons": 30,
    "completedLessons": 15,
    "signsLearned": 45,
    "totalTime": 9000
  }
}
```

```
POST /progress/update
-------------------------------------
Request:
{
  "userId": "uuid-here",
  "courseId": "basics-101",
  "videoId": "v1"
}

Response:
{
  "success": true,
  "message": "Progress updated"
}
```

### 7.4 Recognition API

```
POST /recognize
-------------------------------------
Request:
{
  "frame": "base64-encoded-image-data"
}

Response:
{
  "success": true,
  "sign": "A",
  "confidence": 0.95,
  "alternatives": [
    { "sign": "E", "confidence": 0.03 },
    { "sign": "S", "confidence": 0.02 }
  ]
}
```

---

## 8. Architecture Diagram

```
+--------------------------------------------------------------------+
|                        LearnSign Architecture                       |
+--------------------------------------------------------------------+

                         USER (Browser)
                              |
                              | HTTPS
                              v
                    +-----------------+
                    |    Frontend     |
                    |   (React/HTML)  |
                    |                 |
                    |  Hosted on:     |
                    |  * GitHub Pages |
                    |  * S3 Static    |
                    |  * Netlify      |
                    +--------+--------+
                             |
                             | REST API Calls
                             v
                    +-----------------+
                    |   API Gateway   |    AWS Service #1
                    |                 |
                    |  Routes:        |
                    |  * /auth/*      |
                    |  * /courses/*   |
                    |  * /progress/*  |
                    |  * /recognize   |
                    +--------+--------+
                             |
                             | Invoke
                             v
            +------------------------------------+
            |        AWS Lambda Functions         |    AWS Service #2
            +------------------------------------+
            |                                    |
            |  +----------+    +----------+      |
            |  |  Auth    |    | Courses  |      |
            |  |Functions |    |Functions |      |
            |  +----------+    +----------+      |
            |                                    |
            |  +----------+    +----------+      |
            |  |Progress  |    |Recognition|     |
            |  |Functions |    | Function  |     |
            |  +----------+    +----------+      |
            |                                    |
            +---+----------+----------+---------+
                |          |          |
                v          v          v
          +---------+ +------+ +----------+
          |DynamoDB | |  S3  | |SageMaker |
          |         | |      | |    OR    |
          | Tables: | |Videos| | Lambda   |
          | * Users | |      | |  (ML)    |
          | *Courses| |Models| |          |
          |*Progress| |      | |          |
          +---------+ +------+ +----------+
           AWS #3     AWS #4     AWS #5
```

---

## 9. User Stories

### 9.1 Student User Stories

```
US-001: User Registration
-------------------------------------
As a student,
I want to create an account,
So that I can track my learning progress.

Acceptance Criteria:
[x] Can register with email and password
[x] Receive confirmation of successful registration
[x] Automatically logged in after registration
```

```
US-002: Browse Courses
-------------------------------------
As a student,
I want to browse available ISL courses,
So that I can choose what to learn.

Acceptance Criteria:
[x] See list of all available courses
[x] View course descriptions and video counts
[x] Filter courses by difficulty level
```

```
US-003: Watch Video Lessons
-------------------------------------
As a student,
I want to watch ISL video lessons,
So that I can learn new signs.

Acceptance Criteria:
[x] Play/pause video controls
[x] Progress automatically saved
[x] Move to next lesson easily
```

```
US-004: Practice Sign Recognition
-------------------------------------
As a student,
I want to practice ISL signs with my webcam,
So that I can get feedback on my signing accuracy.

Acceptance Criteria:
[x] Webcam access for real-time recognition
[x] Display recognized sign with confidence
[x] Show correct sign if incorrect
```

```
US-005: Track Progress
-------------------------------------
As a student,
I want to see my learning progress,
So that I can stay motivated and track improvement.

Acceptance Criteria:
[x] View completed lessons count
[x] See signs learned counter
[x] View time spent learning
[x] Visual progress charts
```

### 9.2 Parent/Teacher User Stories

```
US-006: Monitor Child Progress
-------------------------------------
As a parent,
I want to see my child's learning progress,
So that I can support their ISL learning journey.

Acceptance Criteria:
[x] View child's completed lessons
[x] See time spent on platform
[x] Receive progress notifications
```

---

## 10. Non-Functional Requirements

### 10.1 Performance Requirements

| Metric | Requirement |
|--------|-------------|
| **API Response Time** | < 500ms for 95% of requests |
| **Video Load Time** | < 3 seconds initial load |
| **Recognition Latency** | < 1 second per frame |
| **Concurrent Users** | Support 100+ simultaneous users |

### 10.2 Security Requirements

```
Security Measures:
-------------------------------------
* Password hashing with bcrypt (cost factor 10)
* JWT tokens for authentication
* HTTPS for all communications
* CORS configuration for API
* Input validation and sanitization
* Rate limiting on API endpoints
```

### 10.3 Scalability Requirements

```
Scalability Features:
-------------------------------------
* Serverless architecture (auto-scaling)
* DynamoDB on-demand capacity
* S3 for unlimited video storage
* Lambda concurrent execution limits
* CloudFront CDN for video delivery (optional)
```

### 10.4 Availability Requirements

| Metric | Requirement |
|--------|-------------|
| **Uptime Target** | 99.9% availability |
| **Data Backup** | Daily DynamoDB backups |
| **Recovery Time** | < 1 hour for critical failures |

---

## 11. Cost Estimate

### 11.1 AWS Free Tier Usage

| Service | Free Tier Limit | Expected Usage | Monthly Cost |
|---------|-----------------|----------------|--------------|
| **Lambda** | 1M requests | 10K requests | $0 |
| **API Gateway** | 1M requests | 10K requests | $0 |
| **DynamoDB** | 25GB + 25 RCU/WCU | 100MB | $0 |
| **S3** | 5GB storage | 2GB videos | $0 |
| **SageMaker** | None | 10 hours | ~$5 |
| **TOTAL** | | | **~$5** |

### 11.2 Post Free Tier Estimate

```
Estimated Monthly Cost (Low Traffic):
-------------------------------------
Lambda:      ~$2/month (100K requests)
API Gateway: ~$3/month (100K requests)
DynamoDB:    ~$5/month (1GB storage)
S3:          ~$5/month (10GB videos)
SageMaker:   ~$10/month (20 hours)
-------------------------------------
TOTAL:       ~$25/month
```

---

## 12. Success Metrics

### 12.1 Key Performance Indicators

```
KPIs for LearnSign:
-------------------------------------
* User Registration Rate: Target 100+ users/month
* Course Completion Rate: Target 60%+
* Recognition Accuracy: Target 85%+
* User Retention Rate: Target 40%+ monthly
* Average Session Duration: Target 15+ minutes
```

### 12.2 Success Criteria

| Criteria | Target | Measurement |
|----------|--------|-------------|
| **Platform Uptime** | 99.9% | AWS CloudWatch |
| **API Response Time** | < 500ms | Performance monitoring |
| **User Satisfaction** | 4+ stars | User feedback |
| **Bug Count** | < 5 critical | Issue tracking |

---

## 13. References

### 13.1 AWS Documentation
- Lambda: https://docs.aws.amazon.com/lambda/
- API Gateway: https://docs.aws.amazon.com/apigateway/
- DynamoDB: https://docs.aws.amazon.com/dynamodb/
- S3: https://docs.aws.amazon.com/s3/
- SageMaker: https://docs.aws.amazon.com/sagemaker/

### 13.2 ISL Resources
- ISLRTC: https://www.islrtc.nic.in/
- MediaPipe: https://mediapipe.dev/

### 13.3 Development Tools
- AWS Console: https://console.aws.amazon.com/
- Postman: https://www.postman.com/
- GitHub: https://github.com/

---
