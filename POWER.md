---
name: "aws-serverless"
displayName: "AWS Serverless"
description: "Build and deploy serverless applications with AWS Lambda, SAM, API Gateway, and comprehensive serverless tooling"
keywords: ["serverless", "lambda", "sam", "api gateway", "aws", "deployment", "cloudformation", "event-driven", "microservices", "backend", "web app", "dynamodb", "kinesis", "sqs", "kafka", "deploy", "cloudwatch", "cold start", "rest api", "s3", "eventbridge"]
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

| File | Description |
|------|-------------|
| `getting-started.md` | Prerequisites validation and first-use walkthrough |
| `sam-project-setup.md` | SAM project initialization, template selection, and development workflow |
| `web-app-deployment.md` | Full-stack deployment patterns with Lambda Web Adapter |
| `event-source-mappings.md` | ESM configuration for DynamoDB, Kinesis, SQS, and Kafka |
| `serverless-optimization.md` | Performance tuning, cost optimization, and monitoring strategies |
| `serverless-troubleshooting.md` | Symptom-based diagnosis and resolution for common issues |

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

### Get ESM Setup Guidance
```
usePower("aws-serverless", "aws-serverless-mcp", "esm_guidance", {
  "event_source": "dynamodb",
  "guidance_type": "setup"
})
```

### Optimize Event Source Mapping
```
usePower("aws-serverless", "aws-serverless-mcp", "esm_optimize", {
  "action": "analyze",
  "event_source": "kinesis",
  "optimization_targets": ["latency", "throughput"]
})
```

### Generate Security Policy for DynamoDB Streams
```
usePower("aws-serverless", "aws-serverless-mcp", "secure_esm_dynamodb_policy", {
  "region": "us-east-1",
  "account": "123456789012",
  "table_name": "MyTable",
  "function_name": "MyProcessorFunction"
})
```

### Get Serverless Templates
```
usePower("aws-serverless", "aws-serverless-mcp", "get_serverless_templates", {
  "template_type": "API",
  "runtime": "python3.12"
})
```

### Get Lambda Event Schemas
```
usePower("aws-serverless", "aws-serverless-mcp", "get_lambda_event_schemas", {
  "event_source": "sqs",
  "runtime": "python"
})
```

## Best Practices

### Project Setup
- Do: Use `sam_init` (`sam init`) with an appropriate template for your use case
- Do: Select `arm64` architecture for better price-performance unless you need x86-specific dependencies
- Do: Set global defaults in `template.yaml` for timeout, memory, runtime, and tracing
- Don't: Copy-paste SAM templates from the internet without understanding the resource configuration
- Don't: Use the same memory and timeout values for all functions regardless of workload

### Security
- Do: Follow least-privilege IAM policies scoped to specific resources and actions
- Do: Use `secure_esm_*` tools to generate correct IAM policies for event source mappings
- Do: Store secrets in AWS Secrets Manager or SSM Parameter Store, never in environment variables
- Do: Use VPC endpoints instead of NAT Gateways for AWS service access when possible
- Don't: Use wildcard (`*`) resource ARNs or actions in IAM policies
- Don't: Hardcode credentials or secrets in application code or templates

### Performance
- Do: Initialize SDK clients and database connections outside the Lambda handler
- Do: Use `get_metrics` to measure before and after optimization changes
- Do: Right-size memory using AWS Lambda Power Tuning or `get_metrics` data
- Do: Enable X-Ray tracing to identify latency bottlenecks
- Don't: Over-provision memory or concurrency without measuring actual usage
- Don't: Use provisioned concurrency for infrequently invoked functions

### Event Source Mappings
- Do: Use `esm_guidance` to get the correct configuration for your event source type
- Do: Enable `ReportBatchItemFailures` for SQS to avoid reprocessing successful messages
- Do: Enable `BisectBatchOnFunctionError` for stream sources to isolate poison records
- Do: Set `MaximumRetryAttempts` and dead-letter queues for stream-based ESMs
- Don't: Set batch size larger than your function can process within its timeout
- Don't: Use FIFO SQS with large batch sizes when strict ordering is required

### Deployment
- Do: Use `sam_deploy` (`sam deploy --guided`) for first-time setup, then `sam_deploy` (`sam deploy`) for subsequent deploys
- Do: Use `samconfig.toml` with environment-specific sections for multi-environment deployments
- Do: Test locally with `sam_local_invoke` (`sam local invoke`) before deploying to AWS
- Don't: Deploy directly to production without testing in a staging environment
- Don't: Skip the `sam_build` (`sam build`) step before deploying

## Troubleshooting

### AWS CLI or SAM CLI Not Configured
**Error:** `aws: command not found` or `sam: command not found`
**Cause:** AWS CLI or SAM CLI is not installed or not on the system PATH.
**Solution:** Install AWS CLI and SAM CLI, then run `aws --version`, `sam --version`, and `aws sts get-caller-identity` to verify. See the `getting-started.md` steering file for a full prerequisites checklist.

### IAM Permission Denied
**Error:** `AccessDeniedException` or `is not authorized to perform`
**Cause:** The IAM role or user lacks the required permissions for the AWS action.
**Solution:** Check the error message for the specific action and resource ARN. Use the `secure_esm_*` tools to generate correct policies for event source mappings. For SAM deployments, ensure `CAPABILITY_IAM` is set.

### SAM Build Fails
**Error:** `Build Failed` with dependency resolution errors
**Cause:** Missing dependencies, incompatible runtime, or build container issues.
**Solution:** Verify the runtime matches your code. Run `sam_build` (`sam build --use-container`) to build in a Lambda-compatible Docker container. Check that `requirements.txt` or `package.json` is present and valid.

### SAM Deploy Changeset Empty
**Error:** `No changes to deploy. Stack is up to date.`
**Cause:** The template and code haven't changed since the last deployment.
**Solution:** This is informational, not an error. If you expected changes, verify that `sam_build` (`sam build`) ran successfully and check that the correct `samconfig.toml` profile is being used.

### Local Testing Fails
**Error:** Docker-related errors during `sam_local_invoke` (`sam local invoke`)
**Cause:** Docker Desktop is not running or Docker daemon is not accessible.
**Solution:** Start Docker Desktop, wait for it to initialize, then verify with `docker ps`. If using a remote Docker host, check `DOCKER_HOST` environment variable.

### Lambda Timeout
**Error:** `Task timed out after X seconds`
**Cause:** Function execution exceeds the configured timeout, often due to slow external calls, large payloads, or under-provisioned memory.
**Solution:** Increase the `Timeout` value in the SAM template. Increase `MemorySize` (which also increases CPU). Use `get_metrics` to identify whether the function is CPU-bound or IO-bound. For IO-bound functions, check network configuration and connection reuse.

### High Iterator Age on Streams
**Error:** CloudWatch `IteratorAge` metric increasing
**Cause:** The Lambda consumer cannot keep up with the stream's incoming record rate.
**Solution:** Increase `ParallelizationFactor` (up to 10) and `BatchSize`. Use `esm_optimize` to get specific recommendations. Check for poison records causing retries. Scale up Lambda memory for CPU-bound processing.

### Kafka/MSK Connectivity Failure
**Error:** `KAFKA_CONNECTION_ERROR` or authentication failures
**Cause:** Network or authentication misconfiguration between Lambda and the Kafka cluster.
**Solution:** Use `esm_kafka_troubleshoot` with the error message for targeted diagnosis. Verify VPC configuration, security group rules (ports 9092/9094), and IAM or SASL/SCRAM authentication settings.

### CloudFront Cache Not Updating
**Error:** Frontend changes not visible after deployment
**Cause:** CloudFront is serving cached versions of the old assets.
**Solution:** Use `update_webapp_frontend` with cache invalidation enabled. Alternatively, use versioned asset filenames (content hashing) so new deploys use new URLs.

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
2. **Install Docker Desktop**: Required for `sam_local_invoke` (`sam local invoke`) and container-based builds
3. **Verify**: Run `sam --version` and `docker --version`

### MCP Server Configuration
The MCP server is configured in `mcp.json` with the following flags:
- `--allow-write`: Enables write operations (project creation, deployments)
- `--allow-sensitive-data-access`: Enables access to Lambda logs and API Gateway logs
