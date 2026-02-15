# Web Application Deployment Guide

## Overview

Deploy full-stack web applications to AWS Serverless using Lambda Web Adapter, enabling standard web frameworks to run on Lambda without modification.

## Deployment Type Selection

Choose the deployment type based on your application:

| Type | Use Case | What Gets Created |
|------|----------|-------------------|
| **backend** | API services, microservices | Lambda + API Gateway |
| **frontend** | Static sites, SPAs | S3 + CloudFront |
| **fullstack** | Complete web apps | Lambda + API Gateway + S3 + CloudFront |

## Supported Frameworks

### Backend Frameworks
- Express.js, FastAPI, Flask, Spring Boot, ASP.NET Core, Gin

### Frontend Frameworks
- React, Vue.js, Angular, Next.js, Svelte

## Lambda Web Adapter

Lambda Web Adapter allows standard web frameworks to run on Lambda without code changes. The `deploy_webapp` tool automatically configures it.

**How it works:**
- Adds the Lambda Web Adapter layer to your function
- Sets `AWS_LAMBDA_EXEC_WRAPPER` to `/opt/bootstrap`
- Configures the `PORT` environment variable for your application
- Your framework listens on that port as it would normally

**Custom startup**: For applications needing pre-start steps (migrations, config loading), provide a startup script that runs setup commands before `exec`-ing your application.

## Project Structure

### Backend-Only
```
my-backend/
├── src/
│   ├── app.js          # Express application
│   ├── routes/         # API routes
│   └── middleware/     # Custom middleware
├── package.json
└── Dockerfile          # Optional
```

### Frontend-Only
```
my-frontend/
├── dist/               # Built assets
│   ├── index.html
│   └── assets/
└── package.json
```

### Full-Stack
```
my-fullstack-app/
├── frontend/
│   ├── dist/           # Built frontend
│   └── package.json
├── backend/
│   ├── src/
│   └── package.json
└── deployment-config.json
```

## Database Integration

### DynamoDB
- Use PAY_PER_REQUEST billing for unpredictable workloads
- Define partition and sort keys based on access patterns
- Add GSIs for alternate query patterns

### RDS
- Place Lambda in VPC with private subnets for RDS access
- Use VPC endpoints to avoid NAT Gateway costs for AWS service calls
- Set connection pool max to 1 per Lambda instance (concurrency model)
- Store connection strings in Secrets Manager

## API Gateway Configuration

### CORS
Configure CORS on the API Gateway for browser-based frontend access:
- Set `AllowOrigin` to your frontend domain in production (avoid `*`)
- Include necessary headers: `Content-Type`, `Authorization`, `X-Api-Key`
- Set appropriate `MaxAge` for preflight caching

### Custom Domains
Use the `configure_domain` tool to set up custom domains with:
- ACM certificate (must be in us-east-1 for CloudFront)
- Route 53 DNS record
- API Gateway base path mapping

## Environment Management

Use `samconfig.toml` sections for environment-specific configuration:

```toml
[dev.deploy.parameters]
stack_name = "my-app-dev"
parameter_overrides = "Environment=dev"

[prod.deploy.parameters]
stack_name = "my-app-prod"
parameter_overrides = "Environment=prod"
```

Store environment-specific secrets in Secrets Manager or SSM Parameter Store, referenced by environment name.

## Security Considerations

- Follow least-privilege IAM policies: scope actions and resources to what the function needs
- Store database credentials and API keys in Secrets Manager, not environment variables
- Use VPC endpoints for DynamoDB and S3 to avoid NAT Gateway costs and improve security
- Enable CloudTrail for audit logging of API and infrastructure changes

## Frontend Updates

Use `update_webapp_frontend` to push new frontend assets to S3 and optionally invalidate the CloudFront cache. For zero-downtime updates, use content-hashed filenames so old and new assets can coexist.

## Performance Considerations

- **Cold starts**: Use provisioned concurrency for latency-sensitive endpoints. Keep deployment packages small.
- **Connection pooling**: Initialize database connections outside the handler. Set pool max to 1.
- **Caching**: Use CloudFront caching for static assets. Disable caching for API paths.
- **Memory sizing**: Use `get_metrics` to monitor duration and right-size memory allocation.
