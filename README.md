# AWS Serverless Kiro Power

AI-assisted serverless development with AWS Lambda, SAM, CDK, API Gateway, EventBridge, and Step Functions.

## Overview

This power provides comprehensive serverless development guidance with AWS best practices built-in. It integrates with the [AWS Serverless MCP Server](https://awslabs.github.io/mcp/servers/aws-serverless-mcp-server/) to give you access to the complete serverless development lifecycle.

Built as a [Kiro Power](https://kiro.dev/docs/powers/), it provides steering files with workflow guidance, best practices, and troubleshooting references combined with MCP server tools for the full serverless lifecycle.

## Features

### Serverless Application Lifecycle

- **Project Initialization**: Create new SAM or CDK projects with templates and best practices
- **Local Development**: Test Lambda functions locally with Docker containers
- **Build and Deploy**: Compile, package, and deploy to AWS with CloudFormation
- **Monitoring**: Retrieve logs and metrics for debugging and optimization

### Web Application Deployment

- **Full-Stack Apps**: Deploy complete applications with Lambda Web Adapter
- **Response Streaming**: Stream responses from Lambda for LLM output, large payloads, and long-running operations
- **Frontend Assets**: Manage S3 hosting with CloudFront distribution
- **Custom Domains**: Configure Route 53 DNS and ACM certificates

### Event-Driven Architecture

- **Event Source Mappings**: Configure Lambda triggers for S3, SNS, DynamoDB, Kinesis, SQS, Kafka, MQ, DocumentDB
- **EventBridge Integration**: Event bus design, routing rules, Pipes, archive and replay
- **Schema Registry**: Type-safe event handling with schema discovery

### Orchestration and Workflows

- **Lambda Durable Functions**: Multi-step workflows with checkpointing for Python 3.14+ and Node.js 22+
- **Step Functions**: Visual orchestration with 220+ AWS service integrations
- **Patterns**: Sequential, parallel, human-in-the-loop, and saga patterns

### Observability

- **Structured Logging**: JSON logging with Powertools Logger, log level strategy, decorator stacking order
- **Distributed Tracing**: X-Ray with Powertools Tracer, annotations, subsegments, cold start filtering
- **CloudWatch Application Signals**: APM with ADOT, service maps, SLO tracking, anomaly detection
- **Custom Metrics**: Embedded Metric Format (EMF), business KPI vs technical metrics
- **Dashboards and Alarms**: Two-tier dashboard strategy, p90/p99 latency alarms, composite alarms

### Optimization

- **Performance Tuning**: Memory right-sizing, Lambda Power Tuning, cold start optimization, SnapStart, Managed Instances
- **Cost Optimization**: Right-sizing, reserved concurrency, architecture recommendations
- **Troubleshooting**: Symptom-based diagnosis for common serverless issues

## Installation

### Prerequisites

- AWS CLI configured with credentials
- AWS SAM CLI installed
- Docker Desktop (for local testing)
- Python 3.10+ with uv package manager

### Security Note

This power's MCP server is configured with two flags that grant elevated access:

- **`--allow-write`**: Enables write operations such as creating SAM projects, deploying stacks, and modifying AWS resources. Without this flag, the server operates in read-only mode.
- **`--allow-sensitive-data-access`**: Enables access to Lambda function logs and API Gateway logs, which may contain sensitive application data.

To restrict these capabilities, edit `mcp.json` and remove the corresponding flags from the `args` array.

### Install the Power

1. Open Powers panel in Kiro IDE
2. Click "Add Custom Power" and select "Import power from GitHub"
3. Enter: `https://github.com/gunnargrosch/aws-serverless-kiro-power`
4. Press "Enter" to confirm

## What's Included

This power bundles two things:

- **Steering files** (`steering/`): Workflow guidance, best practices, and troubleshooting references that teach Kiro how to build serverless applications effectively
- **An MCP server** ([AWS Serverless MCP Server](https://awslabs.github.io/mcp/servers/aws-serverless-mcp-server/)): Tools for the full serverless lifecycle — project initialization, builds, deployments, testing, observability, and security policy generation

> **Note:** The MCP server is configured to use `awslabs.aws-serverless-mcp-server@latest` so you always get the newest tools and bug fixes. The package is pre-1.0 and under active development. To pin to a specific version for reproducibility, edit the version in `mcp.json`.

## Usage

### Activation Keywords

The power activates when you mention serverless-related topics including:

- `serverless`, `lambda`, `sam`, `cdk`
- `api gateway`, `function url`, `lambda web adapter`
- `dynamodb`, `kinesis`, `sqs`, `kafka`
- `event-driven`, `event source mapping`, `cold start`
- `eventbridge`, `event bus`, `step functions`, `durable functions`, `state machine`

### Example Prompts

**Create a new serverless project:**

```text
I want to build a serverless API for a todo application using Python and AWS Lambda. Can you help me set up a new SAM project?
```

**Deploy a web application:**

```text
I have a React frontend and Express.js backend. Help me deploy this as a full-stack serverless application on AWS.
```

**Set up event processing:**

```text
I need to process messages from an SQS queue with Lambda. Help me configure the event source mapping with batch processing and error handling.
```

**Design an event-driven architecture:**

```text
I'm building an order processing system. Help me set up EventBridge with custom events and multiple consumer services.
```

## Power Structure

```text
aws-serverless-kiro-power/
├── POWER.md                               # Main power configuration
├── mcp.json                               # MCP server configuration
├── steering/                              # Workflow-specific guidance
│   ├── getting-started.md                 # Decision tree: what are you building?
│   ├── sam-project-setup.md               # SAM project initialization and workflow
│   ├── cdk-project-setup.md               # CDK constructs, testing, pipelines, SAM coexistence
│   ├── web-app-deployment.md              # Full-stack deployment with Lambda Web Adapter
│   ├── event-sources.md                   # Lambda event sources: S3, SNS, DynamoDB, Kinesis, SQS, Kafka, MQ, DocumentDB
│   ├── event-driven-architecture.md       # EventBridge, event design, Pipes, schema registry
│   ├── orchestration-and-workflows.md     # Durable Functions, Step Functions, workflow patterns
│   ├── observability.md                   # Logging, tracing, metrics, dashboards, Application Signals
│   ├── optimization.md                    # Performance, cost, streaming, Powertools
│   └── troubleshooting.md                 # Symptom-based diagnosis and resolution
├── README.md                              # This file
└── LICENSE                                # MIT License
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
- [AWS CDK Documentation](https://docs.aws.amazon.com/cdk/v2/guide/)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [Serverless Land](https://serverlessland.com/) - Patterns and examples
- [Kiro Powers Documentation](https://kiro.dev/docs/powers/)
