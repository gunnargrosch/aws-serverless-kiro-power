# SAM Project Setup Guide

## Project Initialization Best Practices

### Choosing the Right Template
- **hello-world**: Basic Lambda function with API Gateway
- **quick-start-web**: Web application with frontend and backend
- **quick-start-cloudformation**: Infrastructure-focused template
- **quick-start-scratch**: Minimal template for custom builds

### Runtime Selection Guidelines
- **Python 3.12**: Best for data processing, ML workloads
- **Node.js 22.x**: Ideal for web APIs, real-time applications
- **Java 21**: Enterprise applications, high-performance computing
- **Go 1.x**: Microservices, high-concurrency applications
- **.NET 8**: Windows-centric applications, enterprise integration

### Architecture Considerations
- **x86_64**: Standard choice, broad compatibility
- **arm64**: Better price-performance ratio, newer Graviton processors

## Project Structure Best Practices

```
my-serverless-app/
├── template.yaml          # SAM template
├── samconfig.toml         # SAM configuration
├── src/                   # Function source code
│   ├── handlers/          # Lambda function handlers
│   ├── layers/            # Shared layers
│   └── utils/             # Utility functions
├── events/                # Test event files
├── tests/                 # Unit and integration tests
└── docs/                  # Documentation
```

## Template Configuration

### Global Settings
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

### Environment-Specific Parameters
```yaml
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]
  
  LogLevel:
    Type: String
    Default: INFO
    AllowedValues: [DEBUG, INFO, WARN, ERROR]
```

## Development Workflow

### 1. Initialize Project
```bash
sam init --runtime python3.12 --dependency-manager pip --app-template hello-world --name my-app
```

### 2. Local Development
```bash
# Build the application
sam build

# Test locally
sam local invoke MyFunction --event events/event.json

# Start local API
sam local start-api --port 3000
```

### 3. Deploy to AWS
```bash
# Deploy with guided setup (first time)
sam deploy --guided

# Subsequent deployments
sam deploy
```

## Configuration Management

### samconfig.toml Structure
```toml
version = 0.1

[default.deploy.parameters]
stack_name = "my-serverless-app"
s3_bucket = "my-deployment-bucket"
s3_prefix = "my-app"
region = "us-east-1"
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Environment=dev"
```

### Environment-Specific Configs
```toml
[dev.deploy.parameters]
stack_name = "my-app-dev"
parameter_overrides = "Environment=dev LogLevel=DEBUG"

[prod.deploy.parameters]
stack_name = "my-app-prod"
parameter_overrides = "Environment=prod LogLevel=WARN"
```

## Security Setup

### IAM Role Template
```yaml
MyFunctionRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
    Policies:
      - PolicyName: MyFunctionPolicy
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - dynamodb:GetItem
                - dynamodb:PutItem
              Resource: !GetAtt MyTable.Arn
```

## Testing Strategy

### Unit Tests
```python
import pytest
from src.handlers.my_handler import lambda_handler

def test_lambda_handler():
    event = {"key": "value"}
    context = {}
    
    response = lambda_handler(event, context)
    
    assert response["statusCode"] == 200
    assert "Hello" in response["body"]
```

### Integration Tests
```bash
# Test deployed function
sam remote invoke MyFunction --stack-name my-app-dev --event events/test.json
```

## Monitoring Setup

### CloudWatch Alarms
```yaml
ErrorAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: !Sub "${AWS::StackName}-errors"
    MetricName: Errors
    Namespace: AWS/Lambda
    Statistic: Sum
    Period: 300
    EvaluationPeriods: 2
    Threshold: 5
    ComparisonOperator: GreaterThanThreshold
```

### X-Ray Tracing
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Tracing: Active
    Environment:
      Variables:
        _X_AMZN_TRACE_ID: !Ref AWS::NoValue
```

This guidance ensures proper SAM project setup with security, monitoring, and best practices built-in.
