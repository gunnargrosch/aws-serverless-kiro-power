# Getting Started with AWS Serverless Development

## Prerequisites

Before using AWS Serverless tools, ensure the following are installed and configured:

- **AWS CLI**: Install and configure with your AWS credentials
  - Verify with: `aws --version` and `aws sts get-caller-identity`
  - **CRITICAL**: If AWS CLI is not configured, DO NOT proceed with serverless setup
- **AWS SAM CLI**: Install via pip, npm, or package managers
  - Verify with: `sam --version`
- **Docker Desktop**: Required for local testing and container-based builds
  - Verify with: `docker --version`
  - **CRITICAL**: Docker must be running for local Lambda testing

## What This Power Does

This power gives you AI-powered assistance for AWS serverless development. Instead of learning complex CLI commands and configurations, you can simply describe what you want to build in natural language.

### Key Capabilities

**🤖 Intelligent Guidance**
- Ask "Should I use Lambda for my API?" and get detailed analysis
- Get recommendations for SAM vs CDK vs CloudFormation
- Understand when to use different AWS services

**🏗️ Project Management**
- Create new serverless projects with best practices
- Deploy full-stack web applications 
- Set up event-driven architectures

**📊 Monitoring & Debugging**
- Retrieve CloudWatch logs and metrics
- Troubleshoot performance issues
- Optimize costs and performance

**🔧 Event Processing**
- Configure Lambda triggers for DynamoDB, Kinesis, SQS, Kafka
- Optimize Event Source Mapping performance
- Handle schema management for EventBridge

## How to Use This Power

Simply describe what you want to accomplish in natural language:

**Examples of what you can ask:**
- "I want to build a REST API for a todo app using Python"
- "Help me deploy my React frontend with Express backend to AWS"
- "My Lambda function is timing out, can you help debug it?"
- "Set up a Lambda to process DynamoDB stream events"
- "What's the best way to handle authentication in serverless apps?"

The power will automatically:
1. Understand your requirements
2. Use the appropriate AWS tools
3. Follow best practices
4. Provide step-by-step guidance

## Testing Complete

The power has been validated and is ready to use. You can now ask any serverless development questions and the power will provide intelligent assistance using AWS best practices.

**No project creation during testing** - The power only creates actual projects when you explicitly request them in conversation.
