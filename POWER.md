---
name: "aws-serverless"
displayName: "AWS Serverless"
description: "Build and deploy serverless applications with AWS Lambda, SAM, API Gateway, EventBridge, Step Functions, and event-driven architectures"
keywords: ["serverless", "lambda", "sam", "cdk", "api gateway", "aws", "deployment", "cloudformation", "event-driven", "microservices", "backend", "web app", "dynamodb", "kinesis", "sqs", "kafka", "deploy", "cloudwatch", "cold start", "rest api", "s3", "eventbridge", "function url", "step functions", "durable functions", "state machine"]
author: "Gunnar Grosch"
---

# AWS Serverless Power

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

## Available Steering Files

Refer to these supporting files for detailed guidance on specific workflows:

| File | When to Use |
| ---- | ----------- |
| `getting-started.md` | Decision tree: what are you building? Routes to the right template, runtime, and guide |
| `sam-project-setup.md` | SAM project initialization, template selection, development workflow, and testing |
| `cdk-project-setup.md` | CDK as an alternative to SAM: constructs, patterns, testing, and deployment |
| `web-app-deployment.md` | Full-stack deployment patterns with Lambda Web Adapter, authentication, and response streaming |
| `event-sources.md` | Lambda event sources: S3, SNS, DynamoDB, Kinesis, SQS, Kafka, MQ, and DocumentDB |
| `event-driven-architecture.md` | EventBridge rules, event design, choreography vs orchestration, Pipes, schema registry |
| `orchestration-and-workflows.md` | Multi-step workflows with Lambda Durable Functions or Step Functions |
| `optimization.md` | Performance tuning, cost optimization, Lambda SnapStart, Managed Instances, and Powertools |
| `observability.md` | Structured logging, distributed tracing, custom metrics, alarms, dashboards, and Logs Insights |
| `troubleshooting.md` | Symptom-based diagnosis and resolution for common issues |

## Available MCP Servers

### aws-serverless-mcp

**Connection:** Local MCP server via `uvx awslabs.aws-serverless-mcp-server@latest`
**Authorization:** Uses AWS credentials from environment (`AWS_PROFILE`, `AWS_REGION`)

#### SAM CLI Tools

| Tool | Description | Required Parameters | Optional Parameters |
|------|-------------|---------------------|---------------------|
| `sam_init` | Initialize a new SAM project | `project_name`, `runtime`, `project_directory`, `dependency_manager` | `application_template`, `architecture`, `package_type`, `tracing` |
| `sam_build` | Build a SAM application | `project_directory` | `use_container`, `parallel`, `template_file`, `profile`, `region` |
| `sam_deploy` | Deploy to AWS via CloudFormation | `application_name`, `project_directory` | `region`, `profile`, `parameter_overrides`, `capabilities`, `config_env`, `s3_bucket`, `tags` |
| `sam_local_invoke` | Test a Lambda function locally | `project_directory`, `resource_name` | `event_file`, `event_data`, `environment_variables_file`, `docker_network` |
| `sam_logs` | Retrieve CloudWatch logs | - | `stack_name`, `resource_name`, `start_time`, `end_time`, `cw_log_group`, `region` |

#### Web Application Tools

| Tool | Description | Required Parameters | Optional Parameters |
|------|-------------|---------------------|---------------------|
| `deploy_webapp` | Deploy full-stack, frontend, or backend apps | `deployment_type`, `project_name`, `project_root` | `backend_configuration`, `frontend_configuration`, `region` |
| `configure_domain` | Set up custom domain with DNS and certificate | `project_name`, `domain_name` | `region`, `create_certificate`, `create_route53_record` |
| `update_webapp_frontend` | Update frontend assets and optionally invalidate cache | `project_name`, `project_root`, `built_assets_path` | `invalidate_cache`, `region` |

#### Event Source Mapping Tools

| Tool | Description | Required Parameters | Optional Parameters |
|------|-------------|---------------------|---------------------|
| `esm_guidance` | Get ESM setup and configuration guidance | - | `event_source`, `guidance_type`, `networking_question` |
| `esm_optimize` | Get performance tuning and cost optimization | - | `action`, `event_source`, `optimization_targets`, `configs`, `esm_uuid` |
| `esm_kafka_troubleshoot` | Diagnose Kafka/MSK connectivity issues | - | `kafka_type`, `issue_type` |

#### Security Policy Tools

| Tool | Description | Required Parameters | Optional Parameters |
|------|-------------|---------------------|---------------------|
| `secure_esm_msk_policy` | Generate IAM policy for MSK event sources | `region`, `account`, `cluster_name`, `cluster_uuid`, `function_name` | `topic_pattern`, `consumer_group_pattern`, `partition` |
| `secure_esm_sqs_policy` | Generate IAM policy for SQS event sources | `region`, `account`, `queue_name`, `function_name` | `partition` |
| `secure_esm_kinesis_policy` | Generate IAM policy for Kinesis event sources | `region`, `account`, `stream_name`, `function_name` | `partition` |
| `secure_esm_dynamodb_policy` | Generate IAM policy for DynamoDB stream event sources | `region`, `account`, `table_name`, `function_name` | `partition` |

#### Schema and EventBridge Tools

| Tool | Description | Required Parameters | Optional Parameters |
|------|-------------|---------------------|---------------------|
| `search_schema` | Search for event schemas by keyword | `keywords`, `registry_name` | `limit`, `next_token` |
| `describe_schema` | Get full schema definition | `schema_name`, `registry_name` | `schema_version` |
| `list_registries` | Browse available schema registries | - | `scope`, `registry_name_prefix`, `limit` |

#### Guidance and Templates Tools

| Tool | Description | Required Parameters | Optional Parameters |
|------|-------------|---------------------|---------------------|
| `get_lambda_guidance` | Get Lambda suitability analysis for a use case | `use_case` | `include_examples` |
| `get_iac_guidance` | Get IaC framework recommendation | - | `iac_tool`, `include_examples` |
| `get_serverless_templates` | Get example templates from Serverless Land | `template_type` | `runtime` |
| `get_lambda_event_schemas` | Get Lambda event schemas for a source type | `event_source`, `runtime` | - |

#### Observability Tools

| Tool | Description | Required Parameters | Optional Parameters |
|------|-------------|---------------------|---------------------|
| `get_metrics` | Retrieve CloudWatch metrics for serverless resources | `project_name` | `resources`, `start_time`, `end_time`, `function_name`, `distribution_id`, `region`, `period` |

#### Deployment Help Tools

| Tool | Description | Required Parameters | Optional Parameters |
|------|-------------|---------------------|---------------------|
| `webapp_deployment_help` | Get help with web app deployment types | `deployment_type` | - |
| `deploy_serverless_app_help` | Get help with SAM deployment by app type | `application_type` | - |

## Tool Usage Examples

### Get Lambda Guidance

```text
usePower("aws-serverless", "aws-serverless-mcp", "get_lambda_guidance", {
  "use_case": "REST API for todo application"
})
```

### Initialize SAM Project

```text
usePower("aws-serverless", "aws-serverless-mcp", "sam_init", {
  "project_name": "my-app",
  "runtime": "python3.12",
  "project_directory": ".",
  "dependency_manager": "pip"
})
```

### Deploy Web Application

```text
usePower("aws-serverless", "aws-serverless-mcp", "deploy_webapp", {
  "deployment_type": "fullstack",
  "project_name": "my-web-app",
  "project_root": "."
})
```

### Get Performance Metrics

```text
usePower("aws-serverless", "aws-serverless-mcp", "get_metrics", {
  "project_name": "my-app",
  "resources": ["lambda", "apiGateway"]
})
```

### Search Event Schemas

```text
usePower("aws-serverless", "aws-serverless-mcp", "search_schema", {
  "keywords": "aws.s3",
  "registry_name": "aws.events"
})
```

### Get ESM Setup Guidance

```text
usePower("aws-serverless", "aws-serverless-mcp", "esm_guidance", {
  "event_source": "dynamodb",
  "guidance_type": "setup"
})
```

### Optimize Event Source Mapping

```text
usePower("aws-serverless", "aws-serverless-mcp", "esm_optimize", {
  "action": "analyze",
  "event_source": "kinesis",
  "optimization_targets": ["latency", "throughput"]
})
```

### Generate Security Policy for DynamoDB Streams

```text
usePower("aws-serverless", "aws-serverless-mcp", "secure_esm_dynamodb_policy", {
  "region": "us-east-1",
  "account": "123456789012",
  "table_name": "MyTable",
  "function_name": "MyProcessorFunction"
})
```

### Get Serverless Templates

```text
usePower("aws-serverless", "aws-serverless-mcp", "get_serverless_templates", {
  "template_type": "API",
  "runtime": "python3.12"
})
```

### Get Lambda Event Schemas

```text
usePower("aws-serverless", "aws-serverless-mcp", "get_lambda_event_schemas", {
  "event_source": "sqs",
  "runtime": "python"
})
```

## Best Practices

### Project Setup

- Do: Use `sam_init` or `cdk init` with an appropriate template for your use case
- Do: Set global defaults for timeout, memory, runtime, and tracing (`Globals` in SAM, construct props in CDK)
- Do: Use AWS Lambda Powertools for structured logging, tracing, metrics (EMF), idempotency, and batch processing — available for Python, TypeScript, Java, and .NET
- Don't: Copy-paste templates from the internet without understanding the resource configuration
- Don't: Use the same memory and timeout values for all functions regardless of workload

### Security

- Do: Follow least-privilege IAM policies scoped to specific resources and actions
- Do: Use `secure_esm_*` tools to generate correct IAM policies for event source mappings
- Do: Store secrets in AWS Secrets Manager or SSM Parameter Store, never in environment variables
- Do: Use VPC endpoints instead of NAT Gateways for AWS service access when possible
- Do: Enable Amazon GuardDuty Lambda Protection to monitor function network activity for threats (cryptocurrency mining, data exfiltration, C2 callbacks)
- Don't: Use wildcard (`*`) resource ARNs or actions in IAM policies
- Don't: Hardcode credentials or secrets in application code or templates
- Don't: Store user data or sensitive information in module-level variables — execution environments can be reused across different callers

### Idempotency

- Do: Write idempotent function code — Lambda delivers events **at least once**, so duplicate invocations must be safe
- Do: Use the AWS Lambda Powertools Idempotency utility (backed by DynamoDB) for critical operations
- Do: Validate and deduplicate events at the start of the handler before performing side effects
- Don't: Assume an event will only ever be processed once

For topic-specific best practices, see the dedicated guide files in the Available Steering Files table above.

## Lambda Limits Quick Reference

Limits that developers commonly hit:

| Resource | Limit |
| ---------- | ------- |
| Function timeout | 900 seconds (15 minutes) |
| Memory | 128 MB – 10,240 MB |
| 1 vCPU equivalent | 1,769 MB memory |
| Synchronous payload (request + response) | 6 MB each |
| Async invocation payload | 1 MB |
| Streamed response | 200 MB |
| Deployment package (.zip, uncompressed) | 250 MB |
| Deployment package (.zip upload, compressed) | 50 MB |
| Container image | 10 GB |
| Layers per function | 5 |
| Environment variables (aggregate) | 4 KB |
| `/tmp` ephemeral storage | 512 MB – 10,240 MB |
| Account concurrent executions (default) | 1,000 (requestable increase) |
| Burst scaling rate | 1,000 new executions per 10 seconds |

Check Service Quotas for your account limits: `aws lambda get-account-settings`

## Troubleshooting Quick Reference

| Error | Cause | Solution |
| ------- | ------- | ---------- |
| `Build Failed` | Missing dependencies | Run `sam_build` with `use_container: true` |
| `Stack is in ROLLBACK_COMPLETE` | Previous deploy failed | Delete stack with `aws cloudformation delete-stack`, redeploy |
| `IteratorAge` increasing | Stream consumer falling behind | Increase `ParallelizationFactor` and `BatchSize`. Use `esm_optimize` |
| EventBridge events silently dropped | No DLQ, retries exhausted | Add `RetryPolicy` + `DeadLetterConfig` to rule target |
| Step Functions failing silently | No retry on Task state | Add `Retry` with `Lambda.ServiceException`, `Lambda.AWSLambdaException` |
| Durable Function not resuming | Missing IAM permissions | Add `lambda:CheckpointDurableExecution` and `lambda:GetDurableExecutionState` |

For detailed troubleshooting, see `troubleshooting.md`.

## Configuration

### Authentication Setup

This power requires AWS credentials configured on the host machine:

1. **Install AWS CLI**: Follow the [AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
2. **Configure credentials**: Run `aws configure` or set up named profiles in `~/.aws/credentials`
3. **Set environment variables** (if not using the default profile):
   - `AWS_PROFILE` - Named profile to use
   - `AWS_REGION` - Target AWS region
4. **Verify access**: Run `aws sts get-caller-identity` to confirm credentials are valid

### SAM CLI Setup

1. **Install SAM CLI**: Follow the [SAM CLI installation guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
2. **Install Docker Desktop**: Required for `sam_local_invoke` and container-based builds
3. **Verify**: Run `sam --version` and `docker --version`

### MCP Server Configuration

The MCP server is configured in `mcp.json` with the following flags:

- `--allow-write`: Enables write operations (project creation, deployments)
- `--allow-sensitive-data-access`: Enables access to Lambda logs and API Gateway logs

**Version policy:** `mcp.json` uses `awslabs.aws-serverless-mcp-server@latest`. This is intentional — the package is pre-1.0 and under active development, so pinning would miss bug fixes and new tool capabilities. If you need a stable, reproducible setup, pin to a specific version:

```json
"args": ["awslabs.aws-serverless-mcp-server@0.1.17", "--allow-write", "--allow-sensitive-data-access"]
```

Check for new versions with `uvx pip index versions awslabs.aws-serverless-mcp-server`.
