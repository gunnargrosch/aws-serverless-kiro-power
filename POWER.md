---
name: "aws-serverless"
displayName: "AWS Serverless Development"
description: "Build and deploy serverless applications with AWS Lambda, SAM, API Gateway, and comprehensive serverless tooling"
keywords: ["serverless", "lambda", "sam", "api gateway", "aws", "deployment", "cloudformation", "event-driven", "microservices", "backend", "web app"]
author: "Gunnar Grosch"
---

# AWS Serverless Development Power

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

## Available MCP Servers

### aws-serverless-mcp

**Connection:** Local MCP server via uvx
**Authorization:** Uses AWS credentials from environment

## Best Practices

### Project Structure
**Always use SAM templates** for consistent infrastructure:
- Start with `sam init` for new projects
- Use appropriate runtime (python3.12, nodejs22.x, etc.)
- Follow SAM template best practices
- Organize code in logical directories

### Lambda Function Design
**Optimize for performance and cost**:
- Right-size memory allocation (128MB-10GB)
- Use appropriate timeout values
- Implement proper error handling
- Enable X-Ray tracing for observability
- Use environment variables for configuration

### Security
**Follow least privilege principles**:
- Use IAM roles with minimal permissions
- Enable CloudTrail for audit logging
- Use AWS Secrets Manager for sensitive data
- Implement proper input validation
- Enable VPC endpoints for private communication

### Event-Driven Architecture
**For event processing**, use Event Source Mappings:
- Configure appropriate batch sizes
- Use dead letter queues for error handling
- Monitor iterator age for streams
- Implement idempotent processing

## Common Workflows

### Workflow 1: Create and Deploy Serverless API

```javascript
// Step 1: Initialize SAM project
sam_init({
  project_name: "todo-api",
  runtime: "python3.12",
  project_directory: "/path/to/projects",
  dependency_manager: "pip"
});

// Step 2: Build application
sam_build({
  project_directory: "/path/to/projects/todo-api"
});

// Step 3: Deploy to AWS
sam_deploy({
  application_name: "todo-api",
  project_directory: "/path/to/projects/todo-api",
  region: "us-east-1"
});
```

### Workflow 2: Deploy Full-Stack Web Application

```javascript
// Step 1: Deploy complete application
deploy_webapp({
  deployment_type: "fullstack",
  project_name: "my-web-app",
  project_root: "/path/to/web-app",
  backend_configuration: {
    runtime: "nodejs22.x",
    port: 3000,
    entry_point: "src/app.js"
  },
  frontend_configuration: {
    built_assets_path: "./dist"
  }
});

// Step 2: Configure custom domain
configure_domain({
  project_name: "my-web-app",
  domain_name: "myapp.example.com"
});
```

### Workflow 3: Optimize Event Source Mapping

```javascript
// Step 1: Get ESM guidance
esm_guidance({
  event_source: "dynamodb",
  guidance_type: "setup"
});

// Step 2: Analyze performance
esm_optimize({
  action: "analyze",
  optimization_targets: ["throughput", "cost"],
  event_source: "kinesis"
});

// Step 3: Troubleshoot issues
esm_kafka_troubleshoot({
  kafka_type: "msk",
  issue_type: "diagnosis"
});
```

### Workflow 4: Monitor and Debug

```javascript
// Step 1: Get application metrics
get_metrics({
  project_name: "todo-api",
  start_time: "2024-01-01T00:00:00Z",
  end_time: "2024-01-01T23:59:59Z",
  resources: ["lambda", "apiGateway"]
});

// Step 2: Retrieve logs
sam_logs({
  stack_name: "todo-api",
  start_time: "1hour ago",
  end_time: "now"
});

// Step 3: Test locally
sam_local_invoke({
  project_directory: "/path/to/projects/todo-api",
  resource_name: "TodoFunction",
  event_file: "/path/to/event.json"
});
```

## Best Practices Summary

### ✅ Do:
- **Use SAM templates** for infrastructure as code
- **Test locally first** with sam_local_invoke
- **Enable X-Ray tracing** for distributed debugging
- **Implement proper error handling** and retry strategies
- **Monitor CloudWatch metrics** for performance insights
- **Use environment variables** for configuration
- **Right-size Lambda memory** based on workload
- **Use provisioned concurrency** for latency-sensitive functions
- **Implement structured logging** for better observability
- **Follow security best practices** with IAM roles

### ❌ Don't:
- **Hardcode credentials** in Lambda functions
- **Use overly large deployment packages** (impacts cold starts)
- **Ignore CloudWatch alarms** and monitoring
- **Deploy without testing** locally first
- **Use default timeouts** without consideration
- **Forget to clean up** unused resources
- **Skip security reviews** of IAM policies
- **Use synchronous calls** for long-running processes
- **Ignore cost optimization** opportunities
- **Deploy to production** without proper CI/CD

## Configuration

**Authentication Required**: AWS CLI credentials and AWS SAM CLI

**Setup Steps:**
1. Install and configure AWS CLI with credentials
2. Install AWS SAM CLI via pip, npm, or package managers
3. Install Docker Desktop for local testing
4. Configure in Kiro Powers UI when installing this power

**Prerequisites**:
- AWS account with appropriate permissions
- AWS CLI configured with credentials
- AWS SAM CLI installed
- Docker Desktop running (for local testing)

**MCP Configuration:**
```json
{
  "mcpServers": {
    "aws-serverless-mcp": {
      "command": "uvx",
      "args": [
        "awslabs.aws-serverless-mcp-server@latest",
        "--allow-write",
        "--allow-sensitive-data-access"
      ],
      "env": {
        "AWS_PROFILE": "default",
        "AWS_REGION": "us-east-1"
      }
    }
  }
}
```

## Troubleshooting

### Error: "AWS credentials not configured"
**Cause:** Missing or invalid AWS credentials
**Solution:**
1. Run `aws configure` to set up credentials
2. Verify with `aws sts get-caller-identity`
3. Check AWS_PROFILE environment variable
4. Ensure IAM permissions are sufficient

### Error: "SAM CLI not found"
**Cause:** AWS SAM CLI not installed
**Solution:**
1. Install via `pip install aws-sam-cli`
2. Verify with `sam --version`
3. Add to PATH if necessary
4. Restart terminal after installation

### Error: "Docker not running"
**Cause:** Docker Desktop not started
**Solution:**
1. Start Docker Desktop application
2. Verify with `docker --version`
3. Ensure Docker daemon is running
4. Check Docker permissions

### Error: "Lambda function timeout"
**Cause:** Function execution exceeds timeout limit
**Solution:**
1. Increase timeout in SAM template
2. Optimize function performance
3. Check CloudWatch logs for bottlenecks
4. Consider increasing memory allocation

### Error: "Permission denied"
**Cause:** Insufficient IAM permissions
**Solution:**
1. Review IAM role permissions
2. Add required policies for AWS services
3. Check resource-based policies
4. Verify cross-account access if applicable

## Tips

1. **Start with SAM templates** - Use `sam init` for consistent project structure
2. **Test locally first** - Use `sam local invoke` before deploying
3. **Monitor from day one** - Set up CloudWatch alarms early
4. **Use steering files** - Load specific guidance for complex workflows
5. **Follow AWS Well-Architected** - Security, performance, cost optimization
6. **Implement CI/CD** - Automate testing and deployment
7. **Use layers** - Share common dependencies across functions
8. **Enable tracing** - X-Ray provides valuable debugging insights
9. **Right-size resources** - Monitor and adjust based on usage
10. **Stay updated** - Follow AWS serverless best practices

---

**License:** MIT
