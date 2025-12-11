---
name: "aws-serverless"
displayName: "Serverless Development on AWS"
description: "Build and deploy serverless applications with AWS Lambda, SAM, API Gateway, and comprehensive serverless tooling"
keywords: ["serverless", "lambda", "sam", "api gateway", "aws", "deployment", "cloudformation", "event-driven", "microservices", "backend", "web app"]
author: "Gunnar Grosch"
---

# Serverless Development on AWS Power

## Overview

Build and deploy serverless applications with AWS Lambda, SAM, API Gateway, and the complete AWS serverless ecosystem. This power provides access to comprehensive serverless development tools through the AWS Serverless MCP Server, enabling you to build production-ready serverless applications with best practices built-in.

Use SAM CLI for project initialization and deployment, Lambda Web Adapter for web applications, or Event Source Mappings for event-driven architectures. The platform handles infrastructure provisioning, scaling, and monitoring automatically.

**Key capabilities:**
- **SAM CLI Integration**: Initialize, build, deploy, and test serverless applications
- **Web Application Deployment**: Deploy full-stack applications with Lambda Web Adapter
- **Event Source Mappings**: Configure Lambda triggers for DynamoDB, Kinesis, SQS, Kafka
- **Schema Management**: Type-safe EventBridge integration with schema registry
- **Observability**: CloudWatch logs, metrics, and X-Ray tracing
- **Performance Optimization**: Right-sizing, cost optimization, and troubleshooting

**Authentication**: Requires AWS CLI configured with credentials and AWS SAM CLI installed.

## When to Use This Power

- Building new serverless applications with AWS Lambda and SAM
- Deploying full-stack web applications to AWS Serverless
- Setting up event-driven architectures with DynamoDB, Kinesis, SQS, or Kafka
- Optimizing Lambda function performance and cost
- Troubleshooting serverless application issues
- Implementing serverless best practices and security patterns
- Managing CloudWatch logs and metrics for serverless apps

## Available MCP Servers

### aws-serverless-mcp

**Connection:** Local MCP server via uvx
**Authorization:** Uses AWS credentials from environment

## Getting Started

**For new users or testing the power:** Use the getting-started steering file for complete step-by-step guidance on prerequisites, safe testing commands, and development workflow. Access it with `readPowerSteering("aws-serverless", "getting-started.md")`.

**For creating new projects:** The getting-started guide covers both creating new serverless applications and working with existing ones, with proper directory handling and user confirmation.

## Integration Guides

**Event Source Mappings:** For setting up and optimizing Lambda triggers, use the `esm_guidance` MCP tool for framework-agnostic setup, or use `readPowerSteering("aws-serverless", "event-source-mappings.md")` for comprehensive ESM patterns.

**Web Application Deployment:** For deploying full-stack applications, use `readPowerSteering("aws-serverless", "web-app-deployment.md")` for complete deployment patterns with Lambda Web Adapter.

**Performance Optimization:** For optimizing serverless applications, use `readPowerSteering("aws-serverless", "serverless-optimization.md")` for comprehensive performance and cost optimization strategies.

## Using MCP Tools

### Get Lambda Guidance
To check if Lambda is suitable for your use case:
```
usePower("aws-serverless", "aws-serverless-mcp", "get_lambda_guidance", {
  "use_case": "REST API for todo application"
})
```

### Initialize SAM Project
```
usePower("aws-serverless", "aws-serverless-mcp", "sam_init", {
  "project_name": "my-app",
  "runtime": "python3.12",
  "project_directory": ".",
  "dependency_manager": "pip"
})
```

### Deploy Web Application
```
usePower("aws-serverless", "aws-serverless-mcp", "deploy_webapp", {
  "deployment_type": "fullstack",
  "project_name": "my-web-app",
  "project_root": "."
})
```

### Get Performance Metrics
```
usePower("aws-serverless", "aws-serverless-mcp", "get_metrics", {
  "project_name": "my-app",
  "resources": ["lambda", "apiGateway"]
})
```

### Search Event Schemas
```
usePower("aws-serverless", "aws-serverless-mcp", "search_schema", {
  "keywords": "aws.s3",
  "registry_name": "aws.events"
})
```

## Troubleshooting

### Prerequisites Not Met
**Error:** AWS CLI or SAM CLI not configured
**Solution:** Use the getting-started steering file to validate and configure prerequisites properly

### Permission Issues
**Error:** AWS credentials or IAM permissions insufficient
**Solution:** Verify AWS CLI configuration and ensure proper IAM permissions for serverless resources

### Local Testing Fails
**Error:** Docker not running or SAM local invoke fails
**Solution:** Ensure Docker Desktop is running and use the troubleshooting steering file for systematic debugging

## Additional Resources

For detailed development workflow guidance, see the steering files which cover:
- Complete project creation and testing workflows
- Event Source Mapping configuration and optimization
- Web application deployment patterns
- Performance optimization strategies
- Systematic troubleshooting approaches

Use `readPowerSteering("aws-serverless", "getting-started")` for the complete getting started guide.
