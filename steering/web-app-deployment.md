# Web Application Deployment Guide

## Overview
Deploy full-stack web applications to AWS Serverless using Lambda Web Adapter, enabling standard web frameworks to run on Lambda without modification.

## Supported Frameworks

### Backend Frameworks
- **Express.js**: Node.js web application framework
- **FastAPI**: Modern Python web framework
- **Flask**: Lightweight Python web framework
- **Spring Boot**: Java enterprise framework
- **ASP.NET Core**: .NET web framework
- **Gin**: Go web framework

### Frontend Frameworks
- **React**: Component-based UI library
- **Vue.js**: Progressive JavaScript framework
- **Angular**: Full-featured TypeScript framework
- **Next.js**: React with SSR/SSG capabilities
- **Svelte**: Compile-time optimized framework

## Deployment Types

### Backend-Only Deployment
Deploy API services and backend logic to Lambda:

```yaml
# Project structure
my-backend/
├── src/
│   ├── app.js          # Express application
│   ├── routes/         # API routes
│   └── middleware/     # Custom middleware
├── package.json        # Dependencies
└── Dockerfile          # Optional container config
```

**Configuration**:
```json
{
  "deployment_type": "backend",
  "backend_configuration": {
    "runtime": "nodejs22.x",
    "port": 3000,
    "entry_point": "src/app.js",
    "memory_size": 512,
    "timeout": 30,
    "environment": {
      "NODE_ENV": "production",
      "DATABASE_URL": "${DATABASE_URL}"
    }
  }
}
```

### Frontend-Only Deployment
Deploy static assets to S3 with CloudFront:

```yaml
# Project structure
my-frontend/
├── dist/               # Built assets
│   ├── index.html
│   ├── assets/
│   └── static/
└── package.json
```

**Configuration**:
```json
{
  "deployment_type": "frontend",
  "frontend_configuration": {
    "built_assets_path": "./dist",
    "index_document": "index.html",
    "error_document": "error.html"
  }
}
```

### Full-Stack Deployment
Deploy both frontend and backend together:

```yaml
# Project structure
my-fullstack-app/
├── frontend/
│   ├── dist/           # Built frontend
│   └── package.json
├── backend/
│   ├── src/
│   └── package.json
└── deployment-config.json
```

## Lambda Web Adapter Integration

### Automatic Integration
The deployment tool automatically configures Lambda Web Adapter:

```yaml
# Generated SAM template
MyWebApp:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: backend/
    Handler: run.sh
    Runtime: nodejs22.x
    Layers:
      - arn:aws:lambda:us-east-1:753240598075:layer:LambdaAdapterLayerX86:22
    Environment:
      Variables:
        AWS_LAMBDA_EXEC_WRAPPER: /opt/bootstrap
        PORT: 3000
```

### Custom Startup Scripts
For complex applications, provide custom startup logic:

```bash
#!/bin/bash
# startup.sh
export NODE_ENV=production
npm run migrate
exec node src/app.js
```

## Database Integration

### DynamoDB Configuration
```json
{
  "database_configuration": {
    "tables": [
      {
        "name": "Users",
        "partition_key": "userId",
        "sort_key": "createdAt",
        "billing_mode": "PAY_PER_REQUEST"
      },
      {
        "name": "Posts",
        "partition_key": "postId",
        "global_secondary_indexes": [
          {
            "name": "UserIndex",
            "partition_key": "userId",
            "sort_key": "createdAt"
          }
        ]
      }
    ]
  }
}
```

### RDS Integration
```yaml
# VPC configuration for RDS access
VpcConfig:
  SecurityGroupIds:
    - !Ref DatabaseSecurityGroup
  SubnetIds:
    - !Ref PrivateSubnet1
    - !Ref PrivateSubnet2

Environment:
  Variables:
    DATABASE_URL: !Sub "postgresql://${DBUsername}:${DBPassword}@${DBEndpoint}:5432/${DBName}"
```

## API Gateway Configuration

### CORS Setup
```yaml
ApiGatewayApi:
  Type: AWS::Serverless::Api
  Properties:
    StageName: prod
    Cors:
      AllowMethods: "'GET,POST,PUT,DELETE,OPTIONS'"
      AllowHeaders: "'Content-Type,X-Amz-Date,Authorization,X-Api-Key'"
      AllowOrigin: "'*'"
      MaxAge: "'600'"
```

### Custom Domain Integration
```yaml
DomainName:
  Type: AWS::ApiGateway::DomainName
  Properties:
    DomainName: !Ref CustomDomain
    CertificateArn: !Ref SSLCertificate
    SecurityPolicy: TLS_1_2

BasePathMapping:
  Type: AWS::ApiGateway::BasePathMapping
  Properties:
    DomainName: !Ref DomainName
    RestApiId: !Ref ApiGatewayApi
    Stage: prod
```

## CloudFront Configuration

### Distribution Setup
```yaml
CloudFrontDistribution:
  Type: AWS::CloudFront::Distribution
  Properties:
    DistributionConfig:
      Origins:
        - Id: S3Origin
          DomainName: !GetAtt S3Bucket.RegionalDomainName
          S3OriginConfig:
            OriginAccessIdentity: !Sub "origin-access-identity/cloudfront/${OriginAccessIdentity}"
        - Id: ApiOrigin
          DomainName: !Sub "${ApiGatewayApi}.execute-api.${AWS::Region}.amazonaws.com"
          CustomOriginConfig:
            HTTPPort: 443
            OriginProtocolPolicy: https-only
      DefaultCacheBehavior:
        TargetOriginId: S3Origin
        ViewerProtocolPolicy: redirect-to-https
        CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6  # Managed-CachingOptimized
      CacheBehaviors:
        - PathPattern: "/api/*"
          TargetOriginId: ApiOrigin
          ViewerProtocolPolicy: https-only
          CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad  # Managed-CachingDisabled
```

### Cache Optimization
```yaml
# Static assets caching
StaticAssetsBehavior:
  PathPattern: "/static/*"
  TargetOriginId: S3Origin
  ViewerProtocolPolicy: https-only
  CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6
  ResponseHeadersPolicyId: 67f7725c-6f97-4210-82d7-5512b31e9d03  # Managed-SecurityHeadersPolicy

# API no-cache behavior
ApiBehavior:
  PathPattern: "/api/*"
  TargetOriginId: ApiOrigin
  ViewerProtocolPolicy: https-only
  CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad  # Managed-CachingDisabled
```

## Environment Management

### Multi-Environment Setup
```yaml
# samconfig.toml
[dev.deploy.parameters]
stack_name = "my-app-dev"
parameter_overrides = "Environment=dev DatabaseUrl=dev-db-url"

[staging.deploy.parameters]
stack_name = "my-app-staging"
parameter_overrides = "Environment=staging DatabaseUrl=staging-db-url"

[prod.deploy.parameters]
stack_name = "my-app-prod"
parameter_overrides = "Environment=prod DatabaseUrl=prod-db-url"
```

### Environment Variables
```json
{
  "environment": {
    "NODE_ENV": "production",
    "API_BASE_URL": "https://api.example.com",
    "DATABASE_URL": "${DATABASE_URL}",
    "JWT_SECRET": "${JWT_SECRET}",
    "REDIS_URL": "${REDIS_URL}"
  }
}
```

## Security Best Practices

### IAM Roles and Policies
```yaml
WebAppExecutionRole:
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
      - arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole
    Policies:
      - PolicyName: DynamoDBAccess
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - dynamodb:GetItem
                - dynamodb:PutItem
                - dynamodb:UpdateItem
                - dynamodb:DeleteItem
                - dynamodb:Query
                - dynamodb:Scan
              Resource: 
                - !GetAtt UsersTable.Arn
                - !Sub "${UsersTable.Arn}/index/*"
```

### Secrets Management
```yaml
DatabaseSecret:
  Type: AWS::SecretsManager::Secret
  Properties:
    Description: Database credentials
    GenerateSecretString:
      SecretStringTemplate: '{"username": "admin"}'
      GenerateStringKey: 'password'
      PasswordLength: 32
      ExcludeCharacters: '"@/\'

# Reference in Lambda
Environment:
  Variables:
    DB_SECRET_ARN: !Ref DatabaseSecret
```

## Monitoring and Logging

### CloudWatch Integration
```yaml
LogGroup:
  Type: AWS::Logs::LogGroup
  Properties:
    LogGroupName: !Sub "/aws/lambda/${FunctionName}"
    RetentionInDays: 14

# Application Insights
ApplicationInsights:
  Type: AWS::ApplicationInsights::Application
  Properties:
    ResourceGroupName: !Ref ResourceGroup
    AutoConfigurationEnabled: true
```

### Custom Metrics
```javascript
// In your application code
const AWS = require('aws-sdk');
const cloudwatch = new AWS.CloudWatch();

async function publishCustomMetric(metricName, value) {
  await cloudwatch.putMetricData({
    Namespace: 'MyApp/Business',
    MetricData: [{
      MetricName: metricName,
      Value: value,
      Unit: 'Count',
      Timestamp: new Date()
    }]
  }).promise();
}
```

## Performance Optimization

### Cold Start Mitigation
```yaml
# Provisioned concurrency
ProvisionedConcurrencyConfig:
  Type: AWS::Lambda::ProvisionedConcurrencyConfig
  Properties:
    FunctionName: !Ref WebAppFunction
    Qualifier: !GetAtt WebAppVersion.Version
    ProvisionedConcurrencyLimit: 5

# Smaller deployment packages
WebAppFunction:
  Type: AWS::Serverless::Function
  Properties:
    PackageType: Zip
    CodeUri: 
      Bucket: !Ref DeploymentBucket
      Key: optimized-package.zip
```

### Connection Pooling
```javascript
// Database connection pooling
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 1, // Important: Lambda concurrency model
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

module.exports = { pool };
```

## Deployment Automation

### CI/CD Pipeline
```yaml
# GitHub Actions example
name: Deploy Web App
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Build Frontend
        run: |
          cd frontend
          npm ci
          npm run build
      
      - name: Deploy to AWS
        run: |
          sam build
          sam deploy --no-confirm-changeset
```

This guide provides comprehensive coverage of web application deployment patterns, security considerations, and optimization strategies for AWS Serverless platforms.
