# LearnSign- Design Document

## 1. Overview

### 1.1 Purpose
Simple technical design for LearnSign - an ISL learning platform built by students for a hackathon. Focus on **5 core AWS services** and practical implementation.

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
+------------------------------------------------------------+
|                  LearnSign Architecture                    |
|                    (Student Project)                       |
+------------------------------------------------------------+

                    INTERNET
                       |
                       v
              +----------------+
              |   Frontend     |
              |  React/HTML    |
              |                |
              |  Hosted on:    |
              |  - GitHub      |
              |  - S3 Static   |
              |  - Netlify     |
              +-------+--------+
                      |
                      | HTTPS (REST API)
                      v
              +----------------+
              |  API Gateway   |   AWS Service #1
              |                |
              |  /auth/*       |
              |  /courses/*    |
              |  /progress/*   |
              |  /recognize    |
              +-------+--------+
                      |
                      | Invoke
                      v
      +-----------------------------------+
      |       AWS Lambda Functions        |   AWS Service #2
      +-----------------------------------+
      |                                   |
      |  +----------+  +----------+       |
      |  |  Auth    |  | Courses  |       |
      |  | Functions|  | Functions|       |
      |  +----------+  +----------+       |
      |                                   |
      |  +----------+  +----------+       |
      |  |Progress  |  |Recognition|      |
      |  | Functions|  | Function  |      |
      |  +----------+  +----------+       |
      |                                   |
      +--+----------+----------+---------+
         |          |          |
         |          |          |
         v          v          v
   +---------+ +------+ +----------+
   |DynamoDB | |  S3  | |SageMaker |
   |         | |      | |    OR    |
   | Tables: | |Videos| | Lambda   |
   | *Users  | |      | |  (ML)    |
   | *Courses| |Models| |          |
   |*Progress| |      | |          |
   +---------+ +------+ +----------+
   AWS #3      AWS #4    AWS #5
```

### 2.2 AWS Services Breakdown

| Service | Purpose | Why? | Cost |
|---------|---------|------|------|
| **API Gateway** | REST API endpoints | Easy Lambda integration | Free tier: 1M requests |
| **Lambda** | Backend logic | No servers, auto-scale | Free tier: 1M invocations |
| **DynamoDB** | Database | Fast, serverless NoSQL | Free tier: 25GB |
| **S3** | Video storage | Cheap, reliable | Free tier: 5GB |
| **SageMaker** | ML inference | Easy model deployment | ~$0.50/hour |

---

## 3. Frontend Design

### 3.1 Technology Stack
```
Frontend: React.js (or plain HTML/CSS/JS)
UI Library: TailwindCSS or Bootstrap
Video Player: HTML5 <video> tag
Webcam: navigator.mediaDevices.getUserMedia()
Charts: Chart.js (for dashboard)
```

### 3.2 Pages & Routes

```
/                  --> Landing page
/login             --> Login/Signup page
/dashboard         --> User dashboard
/courses           --> Course listing
/courses/:id       --> Course detail & videos
/practice          --> ISL recognition practice
/translator        --> Search ISL signs
```

### 3.3 Key Components

#### Login Component
```javascript
// LoginPage.jsx
import React, { useState } from 'react';

function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleLogin = async () => {
    const response = await fetch('https://api-gateway-url/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    localStorage.setItem('token', data.token);
    localStorage.setItem('userId', data.userId);
    window.location.href = '/dashboard';
  };
  
  return (
    <div className="login-container">
      <h1>LearnSign Pro - Login</h1>
      <input 
        type="email" 
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input 
        type="password" 
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button onClick={handleLogin}>Login</button>
    </div>
  );
}
```

#### Webcam Recognition Component
```javascript
// ISLRecognition.jsx
import React, { useEffect, useRef, useState } from 'react';

function ISLRecognition() {
  const videoRef = useRef(null);
  const [result, setResult] = useState(null);
  const [isRecognizing, setIsRecognizing] = useState(false);
  
  // Start webcam
  useEffect(() => {
    navigator.mediaDevices.getUserMedia({ video: true })
      .then(stream => {
        videoRef.current.srcObject = stream;
      });
  }, []);
  
  // Capture and send frame
  const recognizeSign = async () => {
    const video = videoRef.current;
    const canvas = document.createElement('canvas');
    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;
    canvas.getContext('2d').drawImage(video, 0, 0);
    
    const frameData = canvas.toDataURL('image/jpeg').split(',')[1];
    
    setIsRecognizing(true);
    const response = await fetch('https://api-gateway-url/recognize', {
      method: 'POST',
      headers: { 
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify({ frame: frameData })
    });
    
    const data = await response.json();
    setResult(data);
    setIsRecognizing(false);
  };
  
  return (
    <div className="recognition-container">
      <h2>ISL Sign Recognition</h2>
      <video ref={videoRef} autoPlay width="640" height="480"></video>
      
      <button onClick={recognizeSign} disabled={isRecognizing}>
        {isRecognizing ? 'Recognizing...' : 'Recognize Sign'}
      </button>
      
      {result && (
        <div className="result">
          <h3>Result: {result.sign}</h3>
          <p>Confidence: {(result.confidence * 100).toFixed(1)}%</p>
        </div>
      )}
    </div>
  );
}
```

#### Course Viewer Component
```javascript
// CoursePage.jsx
import React, { useEffect, useState } from 'react';

function CoursePage({ courseId }) {
  const [course, setCourse] = useState(null);
  const [currentVideo, setCurrentVideo] = useState(0);
  
  useEffect(() => {
    fetch(`https://api-gateway-url/courses/${courseId}`, {
      headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
    })
      .then(res => res.json())
      .then(data => setCourse(data.course));
  }, [courseId]);
  
  const markComplete = async (videoId) => {
    await fetch('https://api-gateway-url/progress/update', {
      method: 'POST',
      headers: { 
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify({
        userId: localStorage.getItem('userId'),
        courseId,
        videoId
      })
    });
  };
  
  if (!course) return <div>Loading...</div>;
  
  return (
    <div className="course-container">
      <h1>{course.title}</h1>
      <video 
        controls 
        width="800"
        src={course.videos[currentVideo].url}
        onEnded={() => markComplete(course.videos[currentVideo].id)}
      ></video>
      
      <div className="video-list">
        {course.videos.map((video, index) => (
          <button 
            key={video.id}
            onClick={() => setCurrentVideo(index)}
            className={index === currentVideo ? 'active' : ''}
          >
            {video.title}
          </button>
        ))}
      </div>
    </div>
  );
}
```

---

## 4. Backend Design (Lambda Functions)

### 4.1 Lambda Function Structure

```
lambda/
+-- auth/
|   +-- login.js
|   +-- register.js
+-- courses/
|   +-- list.js
|   +-- get.js
+-- progress/
|   +-- get.js
|   +-- update.js
+-- recognition/
    +-- recognize.js
```

### 4.2 Authentication Functions

#### Register Function
```javascript
// lambda/auth/register.js
const AWS = require('aws-sdk');
const bcrypt = require('bcryptjs');
const { v4: uuidv4 } = require('uuid');

const dynamodb = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
  try {
    const { email, password, name } = JSON.parse(event.body);
    
    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // Create user
    const user = {
      userId: uuidv4(),
      email,
      password: hashedPassword,
      name,
      createdAt: Date.now()
    };
    
    // Save to DynamoDB
    await dynamodb.put({
      TableName: 'LearnSign-Users',
      Item: user
    }).promise();
    
    return {
      statusCode: 200,
      headers: { 'Access-Control-Allow-Origin': '*' },
      body: JSON.stringify({ 
        success: true, 
        userId: user.userId 
      })
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};
```

#### Login Function
```javascript
// lambda/auth/login.js
const AWS = require('aws-sdk');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const dynamodb = new AWS.DynamoDB.DocumentClient();
const JWT_SECRET = process.env.JWT_SECRET;

exports.handler = async (event) => {
  try {
    const { email, password } = JSON.parse(event.body);
    
    // Find user
    const result = await dynamodb.scan({
      TableName: 'LearnSign-Users',
      FilterExpression: 'email = :email',
      ExpressionAttributeValues: { ':email': email }
    }).promise();
    
    if (result.Items.length === 0) {
      return {
        statusCode: 401,
        body: JSON.stringify({ error: 'Invalid credentials' })
      };
    }
    
    const user = result.Items[0];
    
    // Check password
    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      return {
        statusCode: 401,
        body: JSON.stringify({ error: 'Invalid credentials' })
      };
    }
    
    // Generate JWT
    const token = jwt.sign(
      { userId: user.userId, email: user.email },
      JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    return {
      statusCode: 200,
      headers: { 'Access-Control-Allow-Origin': '*' },
      body: JSON.stringify({ 
        success: true,
        token,
        userId: user.userId,
        name: user.name
      })
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};
```

### 4.3 Course Functions

#### Get Courses
```javascript
// lambda/courses/list.js
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
  try {
    const result = await dynamodb.scan({
      TableName: 'LearnSign-Courses'
    }).promise();
    
    return {
      statusCode: 200,
      headers: { 'Access-Control-Allow-Origin': '*' },
      body: JSON.stringify({ 
        success: true,
        courses: result.Items 
      })
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};
```

#### Get Single Course
```javascript
// lambda/courses/get.js
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
  try {
    const { courseId } = event.pathParameters;
    
    const result = await dynamodb.get({
      TableName: 'LearnSign-Courses',
      Key: { courseId }
    }).promise();
    
    return {
      statusCode: 200,
      headers: { 'Access-Control-Allow-Origin': '*' },
      body: JSON.stringify({ 
        success: true,
        course: result.Item 
      })
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};
```

### 4.4 ISL Recognition Function

```javascript
// lambda/recognition/recognize.js
const AWS = require('aws-sdk');
const sagemaker = new AWS.SageMakerRuntime();

const ENDPOINT_NAME = 'learnsign-isl-model';

exports.handler = async (event) => {
  try {
    const { frame } = JSON.parse(event.body);
    
    // Option 1: Using SageMaker
    const result = await sagemaker.invokeEndpoint({
      EndpointName: ENDPOINT_NAME,
      ContentType: 'application/json',
      Body: JSON.stringify({ frame })
    }).promise();
    
    const prediction = JSON.parse(result.Body.toString());
    
    return {
      statusCode: 200,
      headers: { 'Access-Control-Allow-Origin': '*' },
      body: JSON.stringify({
        success: true,
        sign: prediction.sign,
        confidence: prediction.confidence
      })
    };
    
    // Option 2: If using Lambda with small model (not SageMaker)
    // Load TensorFlow.js model and run inference directly
    // const tf = require('@tensorflow/tfjs-node');
    // const model = await tf.loadLayersModel('file://./model/model.json');
    // const prediction = model.predict(processedFrame);
    
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};
```

### 4.5 Progress Functions

#### Update Progress
```javascript
// lambda/progress/update.js
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
  try {
    const { userId, courseId, videoId } = JSON.parse(event.body);
    
    // Check if progress record exists
    const existing = await dynamodb.query({
      TableName: 'LearnSign-Progress',
      IndexName: 'userId-courseId-index',
      KeyConditionExpression: 'userId = :uid AND courseId = :cid',
      ExpressionAttributeValues: {
        ':uid': userId,
        ':cid': courseId
      }
    }).promise();
    
    if (existing.Items.length > 0) {
      // Update existing
      const progress = existing.Items[0];
      const completed = progress.completedVideos || [];
      if (!completed.includes(videoId)) {
        completed.push(videoId);
      }
      
      await dynamodb.update({
        TableName: 'LearnSign-Progress',
        Key: { progressId: progress.progressId },
        UpdateExpression: 'SET completedVideos = :cv, lastAccessed = :la',
        ExpressionAttributeValues: {
          ':cv': completed,
          ':la': Date.now()
        }
      }).promise();
    } else {
      // Create new
      await dynamodb.put({
        TableName: 'LearnSign-Progress',
        Item: {
          progressId: require('uuid').v4(),
          userId,
          courseId,
          completedVideos: [videoId],
          lastAccessed: Date.now()
        }
      }).promise();
    }
    
    return {
      statusCode: 200,
      headers: { 'Access-Control-Allow-Origin': '*' },
      body: JSON.stringify({ success: true })
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};
```

---

## 5. Database Design (DynamoDB)

### 5.1 Table Schemas

#### Users Table
```
Table Name: LearnSign-Users
Partition Key: userId (String)

Attributes:
- userId: string (PK)
- email: string
- password: string (hashed)
- name: string
- createdAt: number (timestamp)

GSI: email-index (for login lookup)
```

#### Courses Table
```
Table Name: LearnSign-Courses
Partition Key: courseId (String)

Attributes:
- courseId: string (PK)
- title: string
- description: string
- videos: list
  [
    { id: string, title: string, url: string }
  ]
```

#### Progress Table
```
Table Name: LearnSign-Progress
Partition Key: progressId (String)

Attributes:
- progressId: string (PK)
- userId: string
- courseId: string
- completedVideos: list [videoId1, videoId2, ...]
- lastAccessed: number (timestamp)

GSI: userId-courseId-index
```

### 5.2 Sample Data

#### Sample Course
```json
{
  "courseId": "course-basics",
  "title": "ISL Basics",
  "description": "Learn fundamental ISL signs",
  "videos": [
    {
      "id": "v1",
      "title": "Namaste (Hello)",
      "url": "https://learnsign-videos.s3.amazonaws.com/namaste.webm"
    },
    {
      "id": "v2",
      "title": "Alvida (Goodbye)",
      "url": "https://learnsign-videos.s3.amazonaws.com/alvida.webm"
    },
    {
      "id": "v3",
      "title": "Dhanyavad (Thank you)",
      "url": "https://learnsign-videos.s3.amazonaws.com/dhanyavad.webm"
    }
  ]
}
```

---

## 6. API Gateway Configuration

### 6.1 API Routes

```
POST   /auth/register          --> lambda/auth/register
POST   /auth/login             --> lambda/auth/login

GET    /courses                --> lambda/courses/list
GET    /courses/{courseId}     --> lambda/courses/get

GET    /progress/{userId}      --> lambda/progress/get
POST   /progress/update        --> lambda/progress/update

POST   /recognize              --> lambda/recognition/recognize
```

### 6.2 CORS Configuration

```javascript
// Enable CORS for all endpoints
{
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type, Authorization"
}
```

### 6.3 API Gateway Setup (AWS Console)

```bash
1. Go to API Gateway console
2. Create REST API
3. Create resources (/auth, /courses, /progress, /recognize)
4. Add methods (GET, POST) to each resource
5. Connect to Lambda functions
6. Enable CORS
7. Deploy to "prod" stage
8. Copy API endpoint URL
```

---

## 7. ML Model Deployment

### 7.1 Option A: SageMaker Endpoint

```python
# deploy_model.py
import boto3
import sagemaker
from sagemaker.tensorflow import TensorFlowModel

# Upload model to S3
s3_model_path = 's3://learnsign-models/isl-recognition-model.tar.gz'

# Create SageMaker model
model = TensorFlowModel(
    model_data=s3_model_path,
    role='arn:aws:iam::123456789:role/SageMakerRole',
    framework_version='2.12'
)

# Deploy to endpoint
predictor = model.deploy(
    initial_instance_count=1,
    instance_type='ml.t2.medium',  # Cheapest option
    endpoint_name='learnsign-isl-model'
)

print(f"Endpoint deployed: {predictor.endpoint_name}")
```

### 7.2 Option B: Lambda with TensorFlow.js (Simpler!)

```javascript
// If model is small enough (<250MB), run directly in Lambda
const tf = require('@tensorflow/tfjs-node');

let model = null;

exports.handler = async (event) => {
  // Load model once (cold start)
  if (!model) {
    model = await tf.loadLayersModel('file://./model/model.json');
  }
  
  const { frame } = JSON.parse(event.body);
  
  // Preprocess frame
  const tensor = tf.node.decodeImage(Buffer.from(frame, 'base64'));
  const resized = tf.image.resizeBilinear(tensor, [224, 224]);
  const normalized = resized.div(255.0);
  
  // Run inference
  const prediction = model.predict(normalized.expandDims(0));
  const probabilities = await prediction.data();
  
  // Get top prediction
  const maxProb = Math.max(...probabilities);
  const signIndex = probabilities.indexOf(maxProb);
  const signs = ['A', 'B', 'C', /* ... all ISL signs */];
  
  return {
    statusCode: 200,
    body: JSON.stringify({
      sign: signs[signIndex],
      confidence: maxProb
    })
  };
};
```

---

## 8. Deployment Guide

### 8.1 Step-by-Step Deployment

#### Step 1: Set Up DynamoDB Tables
```bash
# Create Users table
aws dynamodb create-table \
  --table-name LearnSign-Users \
  --attribute-definitions AttributeName=userId,AttributeType=S \
  --key-schema AttributeName=userId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# Create Courses table
aws dynamodb create-table \
  --table-name LearnSign-Courses \
  --attribute-definitions AttributeName=courseId,AttributeType=S \
  --key-schema AttributeName=courseId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# Create Progress table
aws dynamodb create-table \
  --table-name LearnSign-Progress \
  --attribute-definitions AttributeName=progressId,AttributeType=S \
  --key-schema AttributeName=progressId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

#### Step 2: Upload Videos to S3
```bash
# Create bucket
aws s3 mb s3://learnsign-videos

# Upload videos
aws s3 cp ./videos/ s3://learnsign-videos/ --recursive

# Make bucket public (for reading)
aws s3api put-bucket-policy --bucket learnsign-videos --policy '{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::learnsign-videos/*"
  }]
}'
```

#### Step 3: Deploy Lambda Functions
```bash
# Install dependencies
cd lambda/auth/login
npm install

# Zip function
zip -r function.zip .

# Create Lambda
aws lambda create-function \
  --function-name learnsign-auth-login \
  --runtime nodejs18.x \
  --role arn:aws:iam::123456789:role/LambdaExecutionRole \
  --handler login.handler \
  --zip-file fileb://function.zip \
  --environment Variables={JWT_SECRET=your-secret-key}

# Repeat for all Lambda functions
```

#### Step 4: Set Up API Gateway
```bash
# Create REST API
aws apigateway create-rest-api --name LearnSignAPI

# Get API ID
API_ID=$(aws apigateway get-rest-apis --query "items[?name=='LearnSignAPI'].id" --output text)

# Create resources and methods (use AWS Console for easier setup)
```

#### Step 5: Deploy Frontend
```bash
# Build React app
npm run build

# Deploy to S3
aws s3 sync build/ s3://learnsign-frontend

# Enable static website hosting
aws s3 website s3://learnsign-frontend/ \
  --index-document index.html \
  --error-document index.html
```

---

## 9. Testing

### 9.1 Test APIs with cURL

```bash
# Test Register
curl -X POST https://your-api-id.execute-api.us-east-1.amazonaws.com/prod/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123","name":"Test User"}'

# Test Login
curl -X POST https://your-api-id.execute-api.us-east-1.amazonaws.com/prod/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123"}'

# Test Get Courses
curl https://your-api-id.execute-api.us-east-1.amazonaws.com/prod/courses \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### 9.2 Test Recognition

```javascript
// Test in browser console
async function testRecognition() {
  const video = document.querySelector('video');
  const canvas = document.createElement('canvas');
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  canvas.getContext('2d').drawImage(video, 0, 0);
  
  const frame = canvas.toDataURL('image/jpeg').split(',')[1];
  
  const response = await fetch('https://your-api/recognize', {
    method: 'POST',
    headers: { 
      'Content-Type': 'application/json',
      'Authorization': 'Bearer ' + localStorage.getItem('token')
    },
    body: JSON.stringify({ frame })
  });
  
  const result = await response.json();
  console.log('Recognized sign:', result);
}
```

---

## 10. Cost Estimate (AWS Free Tier)

| Service | Free Tier | Expected Usage | Cost |
|---------|-----------|----------------|------|
| **Lambda** | 1M requests/month | 10K requests | $0 |
| **API Gateway** | 1M requests/month | 10K requests | $0 |
| **DynamoDB** | 25GB storage + 25 RCU/WCU | 100MB | $0 |
| **S3** | 5GB storage | 2GB videos | $0 |
| **SageMaker** | None | 10 hours testing | ~$5 |
| **TOTAL** | | | **~$5 for hackathon** |

**Note**: After free tier expires, estimated ~$10-20/month for low traffic.

---

## 11. Troubleshooting

### Common Issues

#### 1. Lambda timeout
```javascript
// Increase timeout in Lambda settings
// AWS Console --> Lambda --> Configuration --> Timeout: 30 seconds
```

#### 2. CORS errors
```javascript
// Add CORS headers to all Lambda responses
{
  statusCode: 200,
  headers: {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'Content-Type,Authorization'
  },
  body: JSON.stringify(data)
}
```

#### 3. DynamoDB access denied
```javascript
// Add DynamoDB permissions to Lambda execution role
{
  "Effect": "Allow",
  "Action": [
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:UpdateItem",
    "dynamodb:Query",
    "dynamodb:Scan"
  ],
  "Resource": "arn:aws:dynamodb:*:*:table/LearnSign-*"
}
```

#### 4. S3 videos not loading
```bash
# Make sure bucket policy allows public read
# Add CORS configuration to S3 bucket
aws s3api put-bucket-cors --bucket learnsign-videos --cors-configuration '{
  "CORSRules": [{
    "AllowedOrigins": ["*"],
    "AllowedMethods": ["GET"],
    "AllowedHeaders": ["*"]
  }]
}'
```

---

## 12. Resources

### AWS Documentation
- Lambda: https://docs.aws.amazon.com/lambda/
- API Gateway: https://docs.aws.amazon.com/apigateway/
- DynamoDB: https://docs.aws.amazon.com/dynamodb/
- S3: https://docs.aws.amazon.com/s3/
- SageMaker: https://docs.aws.amazon.com/sagemaker/

### Code References
- AWS SDK for JavaScript: https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/
- TensorFlow.js: https://www.tensorflow.org/js
- MediaPipe: https://mediapipe.dev/

### Tutorials
- Building Serverless APIs: https://aws.amazon.com/serverless/getting-started/
- ML on AWS: https://aws.amazon.com/machine-learning/

---
