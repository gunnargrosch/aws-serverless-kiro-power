# CDK Project Setup Guide

## SAM vs CDK: When to Use Each

Both SAM and CDK synthesize CloudFormation. Choosing between them is a matter of team preference and project context.

| | SAM | CDK |
| - | ----- | ----- |
| **Language** | YAML/JSON (declarative) | TypeScript, Python, Java, Go, C# (imperative) |
| **Learning curve** | Lower — close to CloudFormation | Higher — requires familiarity with a programming language |
| **Local testing** | `sam local invoke` built-in | Requires external tools (Localstack, docker) |
| **Abstraction level** | Thin layer over CloudFormation | Rich L2/L3 constructs handle wiring automatically |
| **Code sharing** | Template fragments only | Full reuse via construct libraries (npm, PyPI) |
| **Loops and conditions** | Limited (`Fn::If`, no loops) | Native language constructs (`for`, `if`, maps) |
| **Testing** | Manual template review | Unit tests with `aws-cdk-lib/assertions` |
| **Best for** | Lambda-centric apps, teams new to IaC | Large teams building reusable infrastructure patterns |

**Choose SAM** when your primary concern is Lambda functions and you want `sam local invoke` and the SAM MCP tools.

**Choose CDK** when you have complex infrastructure, want to write reusable construct libraries, prefer a programming-language interface, or your team already uses CDK elsewhere.

Both tools support the `get_iac_guidance` MCP tool for additional context:

```text
get_iac_guidance(iac_tool: "cdk")
```

---

## Getting Started

### Install and Bootstrap

```bash
npm install -g aws-cdk
cdk --version

# One-time account/region bootstrap (creates CDK toolkit stack)
cdk bootstrap aws://ACCOUNT-ID/REGION
```

### Initialize a New Project

```bash
mkdir my-serverless-app && cd my-serverless-app
cdk init app --language typescript
npm install
```

### Project Structure

```text
my-serverless-app/
├── bin/
│   └── my-serverless-app.ts    # App entry point
├── lib/
│   └── my-serverless-app-stack.ts  # Stack definition
├── lambda/
│   └── handler.ts              # Lambda function code
├── test/
│   └── my-serverless-app.test.ts
├── cdk.context.json            # Committed to git — caches lookups
├── cdk.json                    # CDK config
└── tsconfig.json
```

---

## Construct Levels

CDK has three levels of constructs:

| Level | Description | Example |
| ------- | ------------- | --------- |
| **L1 (Cfn*)** | Direct CloudFormation resource, 1:1 mapping | `CfnFunction`, `CfnTable` |
| **L2** | Opinionated wrapper with sensible defaults and helper methods | `Function`, `Table`, `Queue` |
| **L3 (Patterns)** | Complete patterns that wire multiple resources together | `LambdaRestApi`, `SqsEventSource` |

**Always prefer L2 constructs.** Use L1 only when a feature is missing from the L2. Use L3 patterns as a starting point, but understand what they create.

---

## Lambda Functions

### Node.js — `NodejsFunction`

The `NodejsFunction` construct bundles TypeScript/JavaScript with esbuild automatically. No separate build step needed.

```typescript
import * as cdk from 'aws-cdk-lib';
import { NodejsFunction } from 'aws-cdk-lib/aws-lambda-nodejs';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import { Duration } from 'aws-cdk-lib';

export class MyStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const orderHandler = new NodejsFunction(this, 'OrderHandler', {
      entry: 'lambda/order-handler.ts',   // path to TypeScript entry file
      handler: 'handler',                 // exported function name
      runtime: lambda.Runtime.NODEJS_22_X,
      architecture: lambda.Architecture.ARM_64,  // ~20% cheaper than x86_64
      memorySize: 512,
      timeout: Duration.seconds(30),
      tracing: lambda.Tracing.ACTIVE,
      environment: {
        TABLE_NAME: myTable.tableName,    // reference, not hardcoded string
      },
      bundling: {
        minify: true,
        sourceMap: true,
        externalModules: ['@aws-sdk/*'],  // exclude AWS SDK (provided by runtime)
      },
    });
  }
}
```

**Entry auto-detection:** If `entry` is omitted, CDK looks for `{stack-filename}.{construct-id}.ts` next to the stack file.

**Handler resolution:** `"handler"` (no dot) resolves to `"index.handler"`.

### Python — `PythonFunction` (Alpha)

`PythonFunction` is in a separate alpha package and requires Docker for bundling.

```bash
npm install @aws-cdk/aws-lambda-python-alpha
```

```typescript
import { PythonFunction } from '@aws-cdk/aws-lambda-python-alpha';
import * as lambda from 'aws-cdk-lib/aws-lambda';

const orderHandler = new PythonFunction(this, 'OrderHandler', {
  entry: 'lambda/order-handler',   // directory containing index.py
  index: 'index.py',               // default
  handler: 'handler',              // default
  runtime: lambda.Runtime.PYTHON_3_12,
  architecture: lambda.Architecture.ARM_64,
  memorySize: 512,
  timeout: Duration.seconds(30),
  tracing: lambda.Tracing.ACTIVE,
  environment: {
    TABLE_NAME: myTable.tableName,
  },
});
```

The `entry` directory should contain a `requirements.txt`, `Pipfile`, or `pyproject.toml`. Docker must be running during `cdk synth` and `cdk deploy`.

> **Alpha warning:** `@aws-cdk/aws-lambda-python-alpha` can introduce breaking changes without a major version bump. Pin the exact version and test after upgrades.

### Other Runtimes — Base `Function`

For Java, Go, .NET, or custom runtimes, use the base `Function` construct with `Code.fromAsset()`:

```typescript
import * as lambda from 'aws-cdk-lib/aws-lambda';

const orderHandler = new lambda.Function(this, 'OrderHandler', {
  code: lambda.Code.fromAsset('build/distributions/order-handler.zip'),
  handler: 'com.example.OrderHandler::handleRequest',
  runtime: lambda.Runtime.JAVA_21,
  architecture: lambda.Architecture.ARM_64,
  memorySize: 1024,
  timeout: Duration.seconds(60),
  snapStart: lambda.SnapStartConf.ON_PUBLISHED_VERSIONS,  // Java 11+
  tracing: lambda.Tracing.ACTIVE,
});
```

---

## IAM with `grant*` Methods

CDK L2 constructs expose `grant*` methods that generate least-privilege policies automatically. Prefer these over writing raw `PolicyStatement` objects.

```typescript
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import * as sqs from 'aws-cdk-lib/aws-sqs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as events from 'aws-cdk-lib/aws-events';

// DynamoDB
table.grantReadWriteData(myFunction);
table.grantReadData(readOnlyFunction);

// S3
bucket.grantRead(myFunction);
bucket.grantPut(myFunction);

// SQS
queue.grantSendMessages(myFunction);
queue.grantConsumeMessages(myFunction);

// EventBridge — put events
eventBus.grantPutEventsTo(myFunction);

// Lambda — invoke another function
otherFunction.grantInvoke(myFunction);

// Secrets Manager
secret.grantRead(myFunction);
```

For resources not covered by `grant*`, add a `PolicyStatement` directly:

```typescript
import * as iam from 'aws-cdk-lib/aws-iam';

myFunction.addToRolePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: ['bedrock:InvokeModel'],
  resources: [`arn:aws:bedrock:${this.region}::foundation-model/anthropic.claude-3-sonnet*`],
}));
```

---

## Common Serverless Patterns

### API Gateway HTTP API + Lambda

```typescript
import * as apigwv2 from 'aws-cdk-lib/aws-apigatewayv2';
import { HttpLambdaIntegration } from 'aws-cdk-lib/aws-apigatewayv2-integrations';

const api = new apigwv2.HttpApi(this, 'OrderApi', {
  corsPreflight: {
    allowOrigins: ['https://myapp.example.com'],
    allowMethods: [apigwv2.CorsHttpMethod.GET, apigwv2.CorsHttpMethod.POST],
    allowHeaders: ['Content-Type', 'Authorization'],
  },
});

api.addRoutes({
  path: '/orders',
  methods: [apigwv2.HttpMethod.POST],
  integration: new HttpLambdaIntegration('CreateOrder', createOrderFunction),
});

new cdk.CfnOutput(this, 'ApiUrl', { value: api.apiEndpoint });
```

### Lambda Function URL

```typescript
const fnUrl = myFunction.addFunctionUrl({
  authType: lambda.FunctionUrlAuthType.NONE,   // or AWS_IAM
  invokeMode: lambda.InvokeMode.RESPONSE_STREAM,  // for streaming
  cors: {
    allowedOrigins: ['https://myapp.example.com'],
    allowedMethods: [lambda.HttpMethod.POST],
  },
});

new cdk.CfnOutput(this, 'FunctionUrl', { value: fnUrl.url });
```

### EventBridge Custom Bus + Rule

```typescript
import * as events from 'aws-cdk-lib/aws-events';
import * as targets from 'aws-cdk-lib/aws-events-targets';
import * as sqs from 'aws-cdk-lib/aws-sqs';

const orderEventBus = new events.EventBus(this, 'OrderEventBus', {
  eventBusName: 'order-events',
});

// Archive all events for replay
new events.Archive(this, 'OrderEventArchive', {
  sourceEventBus: orderEventBus,
  archiveName: 'order-events-archive',
  retention: cdk.Duration.days(30),
  eventPattern: { source: ['com.mycompany.orders'] },
});

// DLQ for the rule target
const processDlq = new sqs.Queue(this, 'ProcessOrderDLQ', {
  retentionPeriod: cdk.Duration.days(14),
});

// Rule routing to Lambda
new events.Rule(this, 'OrderPlacedRule', {
  eventBus: orderEventBus,
  eventPattern: {
    source: ['com.mycompany.orders'],
    detailType: ['OrderPlaced'],
  },
  targets: [
    new targets.LambdaFunction(processOrderFunction, {
      retryAttempts: 3,
      maxEventAge: cdk.Duration.hours(1),
      deadLetterQueue: processDlq,
    }),
  ],
});

// Allow publisher to send events to the bus
orderEventBus.grantPutEventsTo(publisherFunction);
```

### DynamoDB Table + Lambda

```typescript
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';

const ordersTable = new dynamodb.Table(this, 'OrdersTable', {
  partitionKey: { name: 'orderId', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'createdAt', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: cdk.RemovalPolicy.RETAIN,   // never delete in production
  pointInTimeRecoverySpecification: {
    pointInTimeRecoveryEnabled: true,
  },
});

ordersTable.addGlobalSecondaryIndex({
  indexName: 'ByUserId',
  partitionKey: { name: 'userId', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'createdAt', type: dynamodb.AttributeType.STRING },
});

// Least-privilege: read-write for the handler
ordersTable.grantReadWriteData(orderHandler);

// Pass table name via environment (never hardcode)
orderHandler.addEnvironment('TABLE_NAME', ordersTable.tableName);
```

### SQS Queue + Lambda ESM

```typescript
import * as sqs from 'aws-cdk-lib/aws-sqs';
import { SqsEventSource } from 'aws-cdk-lib/aws-lambda-event-sources';

const orderQueue = new sqs.Queue(this, 'OrderQueue', {
  visibilityTimeout: cdk.Duration.seconds(90),  // >= function timeout
});

const dlq = new sqs.Queue(this, 'OrderDLQ', {
  retentionPeriod: cdk.Duration.days(14),
});

orderQueue.addDeadLetterQueue({
  queue: dlq,
  maxReceiveCount: 3,
});

orderHandler.addEventSource(new SqsEventSource(orderQueue, {
  batchSize: 10,
  reportBatchItemFailures: true,  // partial batch success
  filters: [
    lambda.FilterCriteria.filter({
      body: { eventType: lambda.FilterRule.isEqual('ORDER_CREATED') },
    }),
  ],
}));
```

---

## Separating Stateful and Stateless Stacks

Stateful resources (databases, queues, S3 buckets, event buses) should be in a separate stack with termination protection. This prevents accidental deletion during routine deployments.

```typescript
// bin/my-app.ts
const app = new cdk.App();

// Stateful stack — deployed once, termination-protected
const stateful = new StatefulStack(app, 'StatefulStack', {
  env: { account: '123456789012', region: 'us-east-1' },
  terminationProtection: true,
});

// Stateless stack — deployed on every code change
const stateless = new StatelessStack(app, 'StatelessStack', {
  env: { account: '123456789012', region: 'us-east-1' },
  // Pass references from stateful stack
  ordersTable: stateful.ordersTable,
  orderEventBus: stateful.orderEventBus,
});
```

```typescript
// lib/stateful-stack.ts
export class StatefulStack extends cdk.Stack {
  public readonly ordersTable: dynamodb.Table;
  public readonly orderEventBus: events.EventBus;

  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    this.ordersTable = new dynamodb.Table(this, 'OrdersTable', {
      // ...
      removalPolicy: cdk.RemovalPolicy.RETAIN,
    });

    this.orderEventBus = new events.EventBus(this, 'OrderEventBus', {
      eventBusName: 'order-events',
    });
  }
}
```

---

## CDK Testing

CDK constructs can be unit tested without deploying. Use `aws-cdk-lib/assertions` with Jest.

```bash
npm install --save-dev jest @types/jest ts-jest
```

```typescript
// test/order-stack.test.ts
import * as cdk from 'aws-cdk-lib';
import { Template, Match } from 'aws-cdk-lib/assertions';
import { OrderStack } from '../lib/order-stack';

describe('OrderStack', () => {
  let template: Template;

  beforeEach(() => {
    const app = new cdk.App();
    const stack = new OrderStack(app, 'TestOrderStack');
    template = Template.fromStack(stack);
  });

  it('creates Lambda function with ARM64 architecture', () => {
    template.hasResourceProperties('AWS::Lambda::Function', {
      Architectures: ['arm64'],
      Runtime: 'nodejs22.x',
    });
  });

  it('grants DynamoDB read-write to order handler', () => {
    template.hasResourceProperties('AWS::IAM::Policy', {
      PolicyDocument: {
        Statement: Match.arrayWith([
          Match.objectLike({
            Action: Match.arrayWith([
              'dynamodb:GetItem',
              'dynamodb:PutItem',
            ]),
          }),
        ]),
      },
    });
  });

  it('has exactly one DynamoDB table', () => {
    template.resourceCountIs('AWS::DynamoDB::Table', 1);
  });

  it('DynamoDB table has retention policy', () => {
    template.hasResource('AWS::DynamoDB::Table', {
      DeletionPolicy: 'Retain',
    });
  });
});
```

**Assert logical IDs of stateful resources** to catch accidental replacements early — renaming a CDK construct ID causes CloudFormation to delete and recreate the resource:

```typescript
it('orders table logical ID is stable', () => {
  const resources = template.findResources('AWS::DynamoDB::Table');
  expect(Object.keys(resources)).toContain('OrdersTable1234ABCD');  // update if intentionally renamed
});
```

---

## Deployment Workflow

```bash
# Synthesize CloudFormation template (runs assertions, no AWS calls)
cdk synth

# Preview changes before deploying
cdk diff

# Deploy all stacks
cdk deploy --all

# Deploy a specific stack
cdk deploy StatelessStack

# Deploy with approval prompt disabled (CI/CD)
cdk deploy --require-approval never

# Destroy a stack (respects RemovalPolicy — RETAIN resources are kept)
cdk destroy StatelessStack
```

---

## CDK Pipelines (CI/CD)

CDK Pipelines is a self-mutating CI/CD pipeline construct built on CodePipeline.

```typescript
import { CodePipeline, CodePipelineSource, ManualApprovalStep, ShellStep } from 'aws-cdk-lib/pipelines';

export class PipelineStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const pipeline = new CodePipeline(this, 'Pipeline', {
      pipelineName: 'MyServerlessPipeline',
      synth: new ShellStep('Synth', {
        input: CodePipelineSource.gitHub('my-org/my-repo', 'main'),
        commands: ['npm ci', 'npm run build', 'npx cdk synth'],
      }),
    });

    // Staging stage
    pipeline.addStage(new MyAppStage(this, 'Staging', {
      env: { account: '111111111111', region: 'us-east-1' },
    }));

    // Production stage with manual approval
    pipeline.addStage(new MyAppStage(this, 'Production', {
      env: { account: '222222222222', region: 'us-east-1' },
    }), {
      pre: [new ManualApprovalStep('PromoteToProduction')],
    });
  }
}
```

The pipeline updates itself: when you push changes to the pipeline stack, the next run applies them before deploying the application.

---

## SAM and CDK Coexistence

Most teams don't switch from SAM to CDK all at once. Both tools produce CloudFormation, so they can coexist in the same project, the same account, and even the same CI/CD pipeline.

### Incremental Migration

The lowest-risk approach is to add CDK alongside SAM rather than replacing it:

1. **Start with one new stack in CDK** — a supporting resource (DynamoDB table, SQS queue, event bus) that your SAM functions reference via SSM parameters or CloudFormation exports
2. **Keep existing SAM stacks untouched** — they continue to deploy via `sam build && sam deploy`
3. **Migrate function stacks gradually** — move functions to CDK one stack at a time as you gain confidence

Cross-stack references between SAM and CDK stacks work the same way as any CloudFormation cross-stack reference: export a value from one stack, import it in the other via `Fn::ImportValue` (SAM) or `cdk.Fn.importValue()` (CDK). Alternatively, write values to SSM Parameter Store and read them from either side.

### Using `sam build` with CDK Templates

SAM CLI can build and locally test functions defined in any CloudFormation template, including one synthesized by CDK:

```bash
# Synthesize the CDK app to cdk.out/
cdk synth

# Use sam build on the synthesized template
sam build --template cdk.out/MyStack.template.json

# Test a function locally
sam local invoke MyFunction --template cdk.out/MyStack.template.json
```

This gives you CDK's construct model for infrastructure while keeping `sam local invoke` for local testing.

### When to Use Which

| Scenario | Recommendation |
| ---------- | ---------------- |
| New Lambda-centric project, small team | SAM — simpler, `sam local invoke` built-in |
| New project with complex infrastructure (VPCs, multiple services) | CDK — richer abstractions, `grant*` methods |
| Existing SAM project, works fine | Keep SAM — migration cost isn't justified |
| Existing SAM project, hitting limits (no loops, hard to share constructs) | Migrate incrementally to CDK |
| Reusable infrastructure patterns shared across teams | CDK construct libraries |
| Need `sam local invoke` + CDK constructs | Hybrid — `cdk synth` + `sam build` on the output |

---

## Best Practices

### Do

- Use TypeScript — type checking catches errors at synthesis time, before any AWS API calls
- Prefer L2 constructs and `grant*` methods over L1 and raw IAM statements
- Never hardcode resource names — always reference generated names (`table.tableName`, `queue.queueUrl`)
- Separate stateful and stateless resources into different stacks; enable termination protection on stateful stacks
- Commit `cdk.context.json` to version control — it caches VPC/AZ lookups for deterministic synthesis
- Write unit tests with `aws-cdk-lib/assertions`; assert logical IDs of stateful resources to detect accidental replacements
- Set `RemovalPolicy.RETAIN` on all databases, S3 buckets, and event buses
- Use `cdk diff` in CI before every deployment to review changes
- Pass all configuration to constructs via props; never read environment variables inside construct constructors

### Don't

- Hardcode account IDs or region strings — use `this.account` and `this.region`
- Use `cdk deploy` directly in production without a pipeline
- Skip `cdk bootstrap` — deployments will fail without the CDK toolkit stack
- Rely on CloudFormation `Parameters` and `Conditions` when you can express the same logic in TypeScript
- Mix `RemovalPolicy.DESTROY` into shared/stateful stacks
- Reference constructs across separate CDK apps using CloudFormation outputs if you can pass object references directly
