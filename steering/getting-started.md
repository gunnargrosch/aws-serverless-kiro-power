# Getting Started with AWS Serverless Development

## Prerequisites

Before using AWS Serverless tools, ensure the following are installed and configured:

- **AWS CLI**: Install and configure with your AWS credentials
  - Verify with: `aws --version` and `aws sts get-caller-identity`
  - **CRITICAL**: If AWS CLI is not configured, DO NOT proceed with serverless setup
- **AWS SAM CLI**: Install via pip, npm, or package managers
  - Verify with: `sam --version`
- **Docker Desktop**: Required for local testing and container-based builds
  - Verify with: `docker --version`
  - **CRITICAL**: Docker must be running for local Lambda testing

## First Use Walkthrough

### Step 1: Validate Prerequisites

Run these commands to confirm your environment is ready:

```bash
aws --version
aws sts get-caller-identity
sam --version
docker --version
```

All four commands must succeed before continuing. If `aws sts get-caller-identity` fails, run `aws configure` to set up credentials.

### Step 2: Create Your First Project

Use the `sam_init` (`sam init`) tool to scaffold a new project:

- Choose a runtime: `python3.12`, `nodejs22.x`, `java21`, or `dotnet8`
- Choose a template: `hello-world` for a basic API, `quick-start-web` for a web app
- Choose architecture: `arm64` for cost savings, `x86_64` for broader compatibility

Ask the user to confirm the project name and target directory before creating.

### Step 3: Build and Test Locally

1. Use `sam_build` (`sam build`) to compile the application
2. Use `sam_local_invoke` (`sam local invoke`) to test the function with a sample event
3. Verify the output matches expected behavior

### Step 4: Deploy to AWS

1. Use `sam_deploy` (`sam deploy`) with `guided: true` for the first deployment
2. Review the changeset before confirming
3. Note the stack outputs (API endpoint URL, function ARN)

### Step 5: Verify and Monitor

1. Test the deployed endpoint or function
2. Use `sam_logs` (`sam logs`) to check CloudWatch logs
3. Use `get_metrics` to review invocation counts and error rates

## Working with Existing Projects

When working with an existing SAM project:

1. Confirm the project has a `template.yaml` or `template.yml` at the root
2. Check for `samconfig.toml` to understand existing deployment configuration
3. Run `sam_build` (`sam build`) to verify the project builds successfully
4. Review the template resources before making changes

## Next Steps

- For web application deployment patterns, see `web-app-deployment.md`
- For event-driven architecture setup, see `event-source-mappings.md`
- For performance tuning, see `serverless-optimization.md`
- For debugging issues, see `serverless-troubleshooting.md`
