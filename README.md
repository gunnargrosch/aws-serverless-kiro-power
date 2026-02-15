# Serverless Development on AWS Kiro Power

A comprehensive Kiro Power for building and deploying serverless applications with AWS Lambda, SAM, API Gateway, and the complete AWS serverless ecosystem.

## Overview

This power provides AI-assisted serverless development with AWS best practices built-in. It integrates with the [AWS Serverless MCP Server](https://awslabs.github.io/mcp/servers/aws-serverless-mcp-server/) to give you access to the complete serverless development lifecycle.

## Features

### Serverless Application Lifecycle
- **Project Initialization**: Create new SAM projects with templates and best practices
- **Local Development**: Test Lambda functions locally with Docker containers
- **Build and Deploy**: Compile, package, and deploy to AWS with CloudFormation
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

### Observability and Optimization
- **CloudWatch Integration**: Logs, metrics, and alarms
- **X-Ray Tracing**: Distributed tracing and performance analysis
- **Cost Optimization**: Right-sizing and efficiency recommendations
- **Security Best Practices**: IAM policies and VPC configurations

## Installation

### Prerequisites
- AWS CLI configured with credentials
- AWS SAM CLI installed
- Docker Desktop (for local testing)
- Python 3.10+ with uv package manager

### Install the Power

1. Open Powers panel in Kiro IDE
2. Click "Add power from GitHub"
3. Enter: `https://github.com/gunnargrosch/serverless-development-on-aws-kiro-power`
4. Click Install

## Usage

### Activation Keywords
The power activates when you mention:
- `serverless`, `lambda`, `sam`, `deploy`
- `api gateway`, `aws`, `cloudformation`
- `dynamodb`, `kinesis`, `sqs`, `kafka`
- `event-driven`, `microservices`, `backend`, `web app`
- `cold start`, `rest api`, `s3`, `eventbridge`, `cloudwatch`

### Example Prompts

**Create a new serverless project:**
```
I want to build a serverless API for a todo application using Python and AWS Lambda. Can you help me set up a new SAM project?
```

**Deploy a web application:**
```
I have a React frontend and Express.js backend. Help me deploy this as a full-stack serverless application on AWS.
```

**Optimize performance:**
```
My Lambda function is experiencing high latency. Can you help me analyze the performance and optimize the configuration?
```

**Troubleshoot issues:**
```
My DynamoDB stream processing is falling behind. The iterator age is increasing. Help me diagnose and fix this issue.
```

## Power Structure

```
serverless-development-on-aws-kiro-power/
├── POWER.md                           # Main power configuration
├── mcp.json                           # MCP server configuration
├── steering/                          # Workflow-specific guidance
│   ├── getting-started.md             # Getting started guide
│   ├── sam-project-setup.md           # SAM project initialization
│   ├── event-source-mappings.md       # ESM configuration and optimization
│   ├── web-app-deployment.md          # Full-stack deployment patterns
│   ├── serverless-optimization.md     # Performance and cost optimization
│   └── serverless-troubleshooting.md  # Issue diagnosis and resolution
└── README.md                          # This file
```

## Available Tools

The power provides access to comprehensive serverless tooling through the AWS Serverless MCP Server:

### SAM CLI Integration
- `sam_init` - Initialize new serverless projects
- `sam_build` - Build applications for deployment
- `sam_deploy` - Deploy to AWS CloudFormation
- `sam_local_invoke` - Test functions locally
- `sam_logs` - Retrieve CloudWatch logs

### Web Application Tools
- `deploy_webapp` - Deploy full-stack applications
- `configure_domain` - Set up custom domains
- `update_webapp_frontend` - Update frontend assets

### Observability Tools
- `get_metrics` - Retrieve performance metrics

### Event Source Mapping Tools
- `esm_guidance` - ESM setup and configuration
- `esm_optimize` - Performance optimization
- `esm_kafka_troubleshoot` - Kafka-specific troubleshooting

### Security Policy Tools
- `secure_esm_msk_policy` - Generate IAM policy for MSK
- `secure_esm_sqs_policy` - Generate IAM policy for SQS
- `secure_esm_kinesis_policy` - Generate IAM policy for Kinesis
- `secure_esm_dynamodb_policy` - Generate IAM policy for DynamoDB streams

### Schema and EventBridge Tools
- `search_schema` - Find event schemas
- `describe_schema` - Get schema definitions
- `list_registries` - Browse schema registries

### Guidance Tools
- `get_lambda_guidance` - Lambda use case recommendations
- `get_iac_guidance` - Infrastructure as Code selection
- `get_serverless_templates` - Example templates from Serverless Land
- `get_lambda_event_schemas` - Lambda event schemas by source type

### Deployment Help Tools
- `webapp_deployment_help` - Web app deployment troubleshooting
- `deploy_serverless_app_help` - SAM deployment troubleshooting

## Contributing

This power is open source and contributions are welcome! Please feel free to:

1. Report issues or suggest improvements
2. Submit pull requests with enhancements
3. Share your serverless patterns and best practices
4. Help improve the documentation

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Related Resources

- [AWS Serverless MCP Server Documentation](https://awslabs.github.io/mcp/servers/aws-serverless-mcp-server/)
- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [Serverless Land](https://serverlessland.com/) - Patterns and examples
- [Kiro Powers Documentation](https://kiro.dev/docs/powers/)

---

Built with ❤️ for the serverless community!
