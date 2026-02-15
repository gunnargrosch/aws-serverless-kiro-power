# SAM Project Setup Guide

## Template Selection

Choose the right template based on your use case:

| Template | Best For |
|----------|----------|
| `hello-world` | Basic Lambda function with API Gateway |
| `quick-start-web` | Web application with frontend and backend |
| `quick-start-cloudformation` | Infrastructure-focused templates |
| `quick-start-scratch` | Minimal template for custom builds |

Use `get_serverless_templates` to browse additional templates from Serverless Land for specific patterns (e.g., API + DynamoDB, step functions, event processing).

## Runtime Selection

| Runtime | Best For |
|---------|----------|
| Python 3.12 | Data processing, ML workloads, scripting |
| Node.js 22.x | Web APIs, real-time applications |
| Java 21 | Enterprise applications, high-performance computing |
| Go 1.x | Microservices, high-concurrency, low-latency |
| .NET 8 | Windows-centric applications, enterprise integration |

**Architecture:** Choose `arm64` (Graviton) for better price-performance unless you have x86-specific dependencies.

## Project Structure

```
my-serverless-app/
├── template.yaml          # SAM template
├── samconfig.toml         # Deployment configuration
├── src/                   # Function source code
│   ├── handlers/          # Lambda function handlers
│   ├── layers/            # Shared layers
│   └── utils/             # Utility functions
├── events/                # Test event files
└── tests/                 # Unit and integration tests
```

## Template Configuration

### Global Settings

Set global defaults in `template.yaml` to apply to all functions:

```yaml
Globals:
  Function:
    Timeout: 30
    MemorySize: 512
    Runtime: python3.12
    Tracing: Active
    Environment:
      Variables:
        LOG_LEVEL: INFO
        POWERTOOLS_SERVICE_NAME: my-service
```

### Environment Parameters

Use CloudFormation parameters to make templates environment-aware:

```yaml
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]
```

Reference `!Ref Environment` in resource names and configuration to differentiate stacks.

## Development Workflow

### 1. Initialize
Use `sam_init` (`sam init`) with chosen runtime, template, and dependency manager.

### 2. Develop
Write handler code in `src/handlers/`. Create test events in `events/`.

### 3. Build
Use `sam_build` (`sam build`) before every deployment. Use `--use-container` for consistent builds with Lambda-compatible dependencies.

### 4. Test Locally
Use `sam_local_invoke` (`sam local invoke`) with a test event to validate before deploying.

### 5. Deploy
Use `sam_deploy` (`sam deploy`) with `guided: true` for the first deploy, which generates `samconfig.toml`. For subsequent deploys, `sam_deploy` (`sam deploy`) reads from `samconfig.toml`.

### 6. Monitor
Use `sam_logs` (`sam logs`) to check function output. Use `get_metrics` to monitor health.

## Configuration Management

### samconfig.toml

Use environment-specific sections:

```toml
[default.deploy.parameters]
stack_name = "my-serverless-app"
region = "us-east-1"
capabilities = "CAPABILITY_IAM"

[dev.deploy.parameters]
stack_name = "my-app-dev"
parameter_overrides = "Environment=dev LogLevel=DEBUG"

[prod.deploy.parameters]
stack_name = "my-app-prod"
parameter_overrides = "Environment=prod LogLevel=WARN"
```

Deploy to a specific environment with `sam_deploy` (`sam deploy --config-env prod`).

## Security

- Follow least-privilege IAM: scope each function's role to only the actions and resources it needs
- Use `AWSLambdaBasicExecutionRole` managed policy for CloudWatch logging
- Add VPC configuration only when the function needs access to VPC resources (RDS, ElastiCache)
- Store secrets in Secrets Manager or SSM Parameter Store

## Testing

- **Unit tests**: Test handler logic with mocked AWS SDK calls
- **Local integration tests**: Use `sam_local_invoke` (`sam local invoke`) with realistic event payloads
- **Remote tests**: Use `sam remote invoke` to test deployed functions (no MCP tool — CLI only)
- **Event files**: Keep sample events in `events/` for repeatable testing
