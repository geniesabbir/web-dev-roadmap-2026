# Day 4: AWS Basics - Amazon Web Services Fundamentals

## Introduction

Amazon Web Services (AWS) is the world's most comprehensive cloud platform. While platforms like Vercel and Railway are great for quick deployments, AWS offers unmatched flexibility and scalability for enterprise applications. Today, you'll learn the essential AWS services and how to deploy applications on Amazon's infrastructure.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand core AWS services
- Deploy static sites with S3 and CloudFront
- Use EC2 for server hosting
- Work with RDS for managed databases
- Understand IAM security basics

---

## AWS Overview

### Core Services Map

```
┌─────────────────────────────────────────────────────────────────┐
│                      AWS Services Overview                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  COMPUTE                    STORAGE                             │
│  ┌─────────────────┐       ┌─────────────────┐                 │
│  │ EC2 - Servers   │       │ S3 - Objects    │                 │
│  │ Lambda - FaaS   │       │ EBS - Blocks    │                 │
│  │ ECS - Containers│       │ EFS - Files     │                 │
│  └─────────────────┘       └─────────────────┘                 │
│                                                                  │
│  DATABASE                   NETWORKING                          │
│  ┌─────────────────┐       ┌─────────────────┐                 │
│  │ RDS - SQL       │       │ VPC - Network   │                 │
│  │ DynamoDB - NoSQL│       │ Route53 - DNS   │                 │
│  │ ElastiCache     │       │ CloudFront - CDN│                 │
│  └─────────────────┘       │ ELB - Load Bal. │                 │
│                            └─────────────────┘                 │
│  DEVELOPER TOOLS            SECURITY                           │
│  ┌─────────────────┐       ┌─────────────────┐                 │
│  │ CodePipeline    │       │ IAM - Identity  │                 │
│  │ CodeBuild       │       │ Cognito - Auth  │                 │
│  │ CodeDeploy      │       │ Secrets Manager │                 │
│  └─────────────────┘       └─────────────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Getting Started

```bash
# 1. Create AWS Account
# https://aws.amazon.com/

# 2. Install AWS CLI
brew install awscli        # macOS
sudo apt install awscli    # Ubuntu

# 3. Configure credentials
aws configure
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region: us-east-1
# Default output format: json

# 4. Verify setup
aws sts get-caller-identity
```

---

## S3 - Simple Storage Service

### Creating a Bucket

```bash
# Create bucket
aws s3 mb s3://my-website-bucket-unique-name

# List buckets
aws s3 ls

# Upload file
aws s3 cp index.html s3://my-bucket/

# Upload directory
aws s3 sync ./dist s3://my-bucket/

# Delete file
aws s3 rm s3://my-bucket/file.txt

# Delete bucket
aws s3 rb s3://my-bucket --force
```

### Static Website Hosting

```json
// bucket-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

```bash
# Enable static hosting
aws s3 website s3://my-bucket --index-document index.html --error-document error.html

# Apply bucket policy
aws s3api put-bucket-policy --bucket my-bucket --policy file://bucket-policy.json

# Website URL
# http://my-bucket.s3-website-us-east-1.amazonaws.com
```

### S3 with CloudFront (CDN)

```bash
# CloudFront distribution provides:
# - HTTPS
# - Global edge locations
# - Caching
# - Custom domain support

# Create via Console or CloudFormation
```

```yaml
# cloudformation-s3-cloudfront.yml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-website-bucket

  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Origins:
          - DomainName: !GetAtt S3Bucket.RegionalDomainName
            Id: S3Origin
            S3OriginConfig:
              OriginAccessIdentity: ''
        Enabled: true
        DefaultRootObject: index.html
        DefaultCacheBehavior:
          TargetOriginId: S3Origin
          ViewerProtocolPolicy: redirect-to-https
          AllowedMethods: [GET, HEAD]
          CachedMethods: [GET, HEAD]
          ForwardedValues:
            QueryString: false
```

---

## EC2 - Elastic Compute Cloud

### Launching an Instance

```bash
# List available AMIs
aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-*"

# Create key pair
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > my-key.pem
chmod 400 my-key.pem

# Create security group
aws ec2 create-security-group --group-name my-sg --description "My security group"

# Add SSH rule
aws ec2 authorize-security-group-ingress --group-name my-sg --protocol tcp --port 22 --cidr 0.0.0.0/0

# Add HTTP rule
aws ec2 authorize-security-group-ingress --group-name my-sg --protocol tcp --port 80 --cidr 0.0.0.0/0

# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key \
  --security-groups my-sg \
  --count 1
```

### Connecting to EC2

```bash
# SSH into instance
ssh -i my-key.pem ec2-user@ec2-xx-xx-xx-xx.compute-1.amazonaws.com

# Or using public IP
ssh -i my-key.pem ec2-user@54.123.45.67
```

### Setting Up Node.js App

```bash
# On EC2 instance

# Install Node.js
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

# Install PM2
sudo npm install -g pm2

# Clone your app
git clone https://github.com/user/app.git
cd app
npm install

# Start with PM2
pm2 start server.js --name "my-app"
pm2 save
pm2 startup

# Install Nginx
sudo yum install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Nginx Configuration

```nginx
# /etc/nginx/conf.d/app.conf
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## RDS - Relational Database Service

### Creating a Database

```bash
# Create PostgreSQL instance
aws rds create-db-instance \
  --db-instance-identifier my-database \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password secretpassword \
  --allocated-storage 20

# Describe instance
aws rds describe-db-instances --db-instance-identifier my-database

# Get endpoint
aws rds describe-db-instances \
  --db-instance-identifier my-database \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text
```

### Connecting to RDS

```javascript
// Using Prisma
// .env
DATABASE_URL="postgresql://admin:secretpassword@my-database.xxxx.us-east-1.rds.amazonaws.com:5432/mydb"

// Using pg
const { Pool } = require('pg');
const pool = new Pool({
  host: 'my-database.xxxx.us-east-1.rds.amazonaws.com',
  port: 5432,
  user: 'admin',
  password: 'secretpassword',
  database: 'mydb',
  ssl: { rejectUnauthorized: false }
});
```

### Security Group for RDS

```bash
# Allow PostgreSQL from EC2 security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-rds-group \
  --protocol tcp \
  --port 5432 \
  --source-group sg-ec2-group
```

---

## Lambda - Serverless Functions

### Creating a Lambda Function

```javascript
// index.js
exports.handler = async (event) => {
  const name = event.queryStringParameters?.name || 'World';

  return {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      message: `Hello, ${name}!`
    })
  };
};
```

```bash
# Zip the function
zip function.zip index.js

# Create function
aws lambda create-function \
  --function-name hello-world \
  --runtime nodejs18.x \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --role arn:aws:iam::123456789:role/lambda-role

# Update function
aws lambda update-function-code \
  --function-name hello-world \
  --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
  --function-name hello-world \
  --payload '{"name": "John"}' \
  response.json
```

### API Gateway Integration

```bash
# Create HTTP API
aws apigatewayv2 create-api \
  --name my-api \
  --protocol-type HTTP

# Add Lambda integration
aws apigatewayv2 create-integration \
  --api-id abc123 \
  --integration-type AWS_PROXY \
  --integration-uri arn:aws:lambda:us-east-1:123456789:function:hello-world

# API URL: https://abc123.execute-api.us-east-1.amazonaws.com
```

---

## IAM - Identity and Access Management

### Users and Roles

```bash
# Create user
aws iam create-user --user-name developer

# Create access keys
aws iam create-access-key --user-name developer

# Attach policy
aws iam attach-user-policy \
  --user-name developer \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

### IAM Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-bucket"
    }
  ]
}
```

### Lambda Execution Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/my-table"
    }
  ]
}
```

---

## Elastic Beanstalk

### Easy Application Deployment

```bash
# Install EB CLI
pip install awsebcli

# Initialize application
eb init -p node.js my-app

# Create environment
eb create my-app-env

# Deploy
eb deploy

# Open in browser
eb open

# View logs
eb logs

# SSH to instance
eb ssh
```

### Configuration

```yaml
# .ebextensions/nodecommand.config
option_settings:
  aws:elasticbeanstalk:container:nodejs:
    NodeCommand: "npm start"
    NodeVersion: 18.x

# .ebextensions/env.config
option_settings:
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
    PORT: 8080
```

---

## Cost Management

### Free Tier Limits

```
┌─────────────────────────────────────────────────────────────────┐
│                    AWS Free Tier (12 months)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  EC2        750 hours/month t2.micro                            │
│  S3         5GB storage, 20K GET, 2K PUT                        │
│  RDS        750 hours/month db.t2.micro                         │
│  Lambda     1M requests/month, 400K GB-seconds                  │
│  CloudFront 1TB data transfer, 10M requests                     │
│  DynamoDB   25GB storage, 25 read/write units                   │
│                                                                  │
│  Always Free:                                                    │
│  • Lambda: 1M requests/month                                    │
│  • DynamoDB: 25GB storage                                       │
│  • SNS: 1M publishes                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Cost Optimization Tips

```bash
# Use spot instances for non-critical workloads
# Right-size instances (use t3.micro/small)
# Set up billing alerts
# Use reserved instances for steady workloads
# Delete unused resources

# Set billing alert
aws cloudwatch put-metric-alarm \
  --alarm-name "Monthly Bill Alert" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 86400 \
  --threshold 50 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:us-east-1:123456789:billing-alerts
```

---

## Deployment Script Example

```bash
#!/bin/bash
# deploy.sh - Deploy to AWS

set -e

# Variables
BUCKET_NAME="my-website-bucket"
DISTRIBUTION_ID="E1234567890ABC"

# Build
echo "Building..."
npm run build

# Sync to S3
echo "Uploading to S3..."
aws s3 sync ./dist s3://$BUCKET_NAME --delete

# Invalidate CloudFront cache
echo "Invalidating cache..."
aws cloudfront create-invalidation \
  --distribution-id $DISTRIBUTION_ID \
  --paths "/*"

echo "Deployment complete!"
```

---

## Practice Exercises

### Exercise 1: Static Website

```bash
# 1. Create S3 bucket
# 2. Enable static hosting
# 3. Upload a React/Next.js build
# 4. Set up CloudFront
# 5. Configure custom domain
```

### Exercise 2: API on Lambda

```bash
# 1. Create Lambda function
# 2. Set up API Gateway
# 3. Connect to DynamoDB
# 4. Deploy and test
```

### Exercise 3: Full Stack on EC2

```bash
# 1. Launch EC2 instance
# 2. Install Node.js, Nginx, PM2
# 3. Deploy Node.js app
# 4. Connect to RDS PostgreSQL
# 5. Configure SSL with Let's Encrypt
```

---

## Quick Reference

### CLI Commands

| Command | Description |
|---------|-------------|
| `aws s3 sync` | Sync files to S3 |
| `aws ec2 run-instances` | Launch EC2 |
| `aws lambda invoke` | Call Lambda |
| `aws rds describe-db-instances` | List RDS |
| `aws iam create-user` | Create user |

### Common Services

| Service | Use Case |
|---------|----------|
| S3 | Static files, backups |
| EC2 | Servers, custom setup |
| RDS | Managed databases |
| Lambda | Serverless functions |
| CloudFront | CDN, caching |
| Route53 | DNS management |

---

## Key Takeaways

1. **S3 + CloudFront** for static sites
2. **EC2** for full server control
3. **Lambda** for serverless APIs
4. **RDS** for managed databases
5. **IAM** for security and access
6. **Watch costs** - set up billing alerts

---

## What's Next?

Tomorrow, we'll learn about **Monitoring** - tracking application health, performance, and errors with logging and alerting systems.
