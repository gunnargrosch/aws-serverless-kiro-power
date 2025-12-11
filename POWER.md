---
name: "aws-serverless"
displayName: "AWS Serverless Development"
description: "Build and deploy serverless applications with AWS Lambda, SAM, API Gateway, and comprehensive serverless tooling"
keywords: ["serverless", "lambda", "sam", "api gateway", "aws", "deployment", "cloudformation", "event-driven", "microservices", "backend", "web app"]
mcpServers: ["aws-serverless-mcp"]
---

# AWS Serverless Development Power

Accelerate serverless application development with AWS Lambda, SAM, and comprehensive tooling for the entire serverless lifecycle.

## Onboarding

### Step 1: Validate Prerequisites
Before using AWS Serverless tools, ensure the following are installed and configured:

- **AWS CLI**: Install and configure with your AWS credentials
  - Verify with: `aws --version` and `aws sts get-caller-identity`
  - **CRITICAL**: If AWS CLI is not configured, DO NOT proceed with serverless setup
- **AWS SAM CLI**: Install via pip, npm, or package managers
  - Verify with: `sam --version`
- **Docker Desktop**: Required for local testing and container-based builds
  - Verify with: `docker --version`
  - **CRITICAL**: Docker must be running for local Lambda testing

### Step 2: Add Serverless Hooks
Add hooks to `.kiro/hooks/` for common serverless workflows:

```json
{
  "enabled": true,
  "name": "Deploy Serverless Application",
  "description": "Build and deploy SAM application to AWS",
  "version": "1",
  "when": {
    "type": "userTriggered"
  },
  "then": {
    "type": "askAgent",
    "prompt": "Build and deploy the SAM application in the current directory using sam_build and sam_deploy tools"
  }
}
```

```json
{
  "enabled": true,
  "name": "Check Application Logs",
  "description": "Retrieve CloudWatch logs for serverless application debugging",
  "version": "1",
  "when": {
    "type": "userTriggered"
  },
  "then": {
    "type": "askAgent",
    "prompt": "Use sam_logs tool to fetch recent logs for troubleshooting"
  }
}
```

## Core Capabilities

### Serverless Application Lifecycle
- **Project Initialization**: Create new SAM projects with templates and best practices
- **Local Development**: Test Lambda functions locally with Docker containers
- **Build & Deploy**: Compile, package, and deploy to AWS with CloudFormation
- **Monitoring**: Retrieve logs and metrics for debugging and optimization

### Web Application Deployment
- **Full-Stack Apps**: Deploy complete applications with Lambda Web Adapter
- **Frontend Assets**: Manage S3 hosting with CloudFront distribution
- **Custom Domains**: Configure Route 53 DNS and ACM certificates
- **Updates**: Hot-swap frontend assets with cache invalidation

### Event-Driven Architecture
- **Event Source Mappings**: Configure Lambda triggers for DynamoDB, Kinesis, SQS, Kafka
- **EventBridge Integration**: Type-safe event handling with schema registry
- **Performance Optimization**: Analyze and optimize ESM configurations
- **Troubleshooting**: Diagnose connectivity and performance issues

## When to Load Steering Files

For complex serverless workflows, load specific guidance:

- Creating new serverless projects → `sam-project-setup.md`
- Building event-driven architectures → `event-source-mappings.md`
- Deploying web applications → `web-app-deployment.md`
- Performance optimization → `serverless-optimization.md`
- Troubleshooting issues → `serverless-troubleshooting.md`

## Best Practices

### Lambda Function Design
- Use appropriate memory allocation (128MB-10GB based on workload)
- Implement proper error handling and retries
- Leverage environment variables for configuration
- Use layers for shared dependencies
- Enable X-Ray tracing for observability

### SAM Template Structure
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Timeout: 30
    MemorySize: 512
    Runtime: python3.12
    Tracing: Active

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.lambda_handler
      Events:
        Api:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

### Security Considerations
- Use IAM roles with least privilege principles
- Enable CloudTrail for audit logging
- Implement proper input validation
- Use AWS Secrets Manager for sensitive data
- Enable VPC endpoints for private communication

### Performance Optimization
- Right-size Lambda memory allocation
- Use provisioned concurrency for consistent latency
- Implement connection pooling for databases
- Optimize cold start times with smaller deployment packages
- Use appropriate batch sizes for event sources

### Cost Management
- Monitor Lambda invocation costs and duration
- Use appropriate timeout values
- Implement efficient retry strategies
- Consider reserved concurrency for predictable workloads
- Regular review of CloudWatch metrics

## Common Patterns

### API Gateway + Lambda
```yaml
ApiGatewayApi:
  Type: AWS::Serverless::Api
  Properties:
    StageName: prod
    Cors:
      AllowMethods: "'GET,POST,PUT,DELETE'"
      AllowHeaders: "'Content-Type,X-Amz-Date,Authorization'"
      AllowOrigin: "'*'"
```

### DynamoDB Integration
```yaml
TodoTable:
  Type: AWS::DynamoDB::Table
  Properties:
    BillingMode: PAY_PER_REQUEST
    AttributeDefinitions:
      - AttributeName: id
        AttributeType: S
    KeySchema:
      - AttributeName: id
        KeyType: HASH
```

### Event Source Mapping
```yaml
MyEventSourceMapping:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt MyKinesisStream.Arn
    FunctionName: !Ref MyFunction
    StartingPosition: LATEST
    BatchSize: 100
```

## Troubleshooting Guide

### Common Issues
1. **Cold Start Latency**: Use provisioned concurrency or optimize package size
2. **Timeout Errors**: Increase timeout or optimize function performance
3. **Permission Errors**: Check IAM roles and resource policies
4. **Memory Issues**: Monitor CloudWatch metrics and adjust allocation
5. **Event Source Problems**: Validate ESM configuration and network connectivity

### Debugging Steps
1. Check CloudWatch logs for error messages
2. Verify IAM permissions and resource policies
3. Test functions locally with SAM CLI
4. Monitor CloudWatch metrics for performance insights
5. Use X-Ray tracing for distributed debugging

This power provides comprehensive serverless development capabilities with AWS best practices built-in.
