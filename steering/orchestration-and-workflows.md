# Orchestration and Workflows Guide

## Choosing an Orchestration Approach

| Approach | Best For | Runtime |
| ---------- | ---------- | --------- |
| **Lambda Durable Functions** | Multi-step business logic and AI/ML pipelines expressed as sequential code, with checkpointing and human-in-the-loop | Python 3.14+, Node.js 22+ |
| **Step Functions Standard** | Cross-service orchestration, long-running auditable workflows, non-idempotent operations | Any (JSON/YAML ASL definition) |
| **Step Functions Express** | High-volume, short-lived event processing, idempotent operations (100k+ exec/sec) | Any |
| **EventBridge + Lambda** | Loosely coupled event-driven choreography with no central coordinator — see [event-driven-architecture.md](event-driven-architecture.md) | Any |

**Key distinction:** Durable Functions keep the workflow logic inside your Lambda code using standard language constructs. Step Functions define the workflow as a separate graph-based state machine that calls Lambda (and 9,000+ API actions across 200+ AWS services). Use Durable Functions when the workflow is tightly coupled to business logic written in Python or Node.js. Use Step Functions when you need visual design, cross-service coordination, or native service integrations without Lambda as an intermediary.

---

## Lambda Durable Functions

Lambda Durable Functions enable resilient multi-step applications that execute for up to one year, with automatic checkpointing, replay, and suspension — without consuming compute charges during wait periods.

### Core Concepts

| Concept | Description |
| --------- | ------------- |
| **Checkpointing** | Progress saved after each step; completed steps are never re-executed |
| **Replay** | On resume, code runs from the start but skips completed steps using cached results |
| **Suspension** | Execution pauses (e.g., awaiting human input) without billing for idle time |
| **Steps** | Units of work wrapped in `context.step()` — checkpointed, retriable |
| **Waits** | Suspension points that pause until an external signal (`callbackId`) is received |

### SDK Packages

| Language | Runtime | Install |
| ---------- | --------- | --------- |
| Python | 3.14+ | `aws-durable-execution-sdk-python` (via `uv`/`pip`) |
| TypeScript/JavaScript | Node.js 22+ | `@aws/durable-execution-sdk-js` (via `npm`) |

### Programming Model — Python

Use the `@durable_execution` decorator. The second parameter becomes a `DurableContext`:

```python
from aws_durable_execution_sdk_python import DurableContext, durable_execution
from aws_durable_execution_sdk_python.config import Duration, WaitForCallbackConfig

@durable_execution
def handler(event: dict, context: DurableContext):
    # Each step is checkpointed — safe to retry or resume from here
    result_a = context.step(
        lambda _: call_first_service(event["input"]),
        "step-a",
    )
    result_b = context.step(
        lambda _: call_second_service(result_a),
        "step-b",
    )
    return result_b
```

**Human-in-the-loop:**

```python
approval = context.wait_for_callback(
    lambda callback_id, _: send_approval_request(callback_id, result_a),
    "await-approval",
    WaitForCallbackConfig(timeout=Duration.from_days(7)),
)
```

**Parallel execution:**

```python
map_result = context.map(
    items,
    lambda ctx, item, idx, all_items: process_item(item),
    "parallel-processing",
)
results = map_result.get_results()
```

**Nested child context** (for agent tool loops):

```python
tool_result = context.run_in_child_context(
    lambda child_ctx: tool.execute(input, child_ctx),
    f"tool:{tool_name}",
)
```

### Programming Model — TypeScript

Use the `withDurableExecution` higher-order function:

```typescript
import { type DurableContext, withDurableExecution } from "@aws/durable-execution-sdk-js";

export const handler = withDurableExecution(
    async (event: EventType, context: DurableContext) => {
        const resultA = await context.step("step-a", async () => {
            return await callFirstService(event.input);
        });
        const resultB = await context.step("step-b", async () => {
            return await callSecondService(resultA);
        });
        return resultB;
    }
);
```

**Human-in-the-loop:**

```typescript
const approval = await context.waitForCallback<string>(
    "await-approval",
    async (callbackId) => { sendApprovalRequest(callbackId, resultA); },
    { timeout: { days: 7 } },
);
```

**Parallel execution:**

```typescript
const mapResult = await context.map(
    "parallel-processing",
    items,
    async (ctx, item, index) => processItem(item),
    { itemNamer: (item, i) => `item-${i}` },
);
const results = mapResult.getResults();
```

**Nested child context:**

```typescript
const toolResult = await context.runInChildContext(
    `tool:${toolName}`,
    async (childContext) => tool.execute(input, childContext),
);
```

### Python vs TypeScript API Differences

| Feature | Python | TypeScript |
| --------- | -------- | ----------- |
| Handler wrapping | `@durable_execution` decorator | `withDurableExecution(async fn)` |
| Step method signature | `context.step(func, name)` | `await context.step(name, asyncFunc)` |
| Wait for callback | `context.wait_for_callback(func, name, config)` | `await context.waitForCallback(name, func, options)` |
| Map method signature | `context.map(items, func, name)` | `await context.map(name, items, asyncFunc, options)` |
| Map results | `result.get_results()` | `result.getResults()` |
| Child context | `context.run_in_child_context(func, name)` | `await context.runInChildContext(name, asyncFunc)` |

### SAM Template Configuration

```yaml
Globals:
  Function:
    Runtime: python3.14        # or nodejs22.x
    Architectures: [arm64]
    Timeout: 30
    MemorySize: 256

Resources:
  MyDurableFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src.workflow.handler
      Role: !GetAtt DurableFunctionRole.Arn
      DurableConfig:
        ExecutionTimeout: 900          # Max active compute seconds per execution
        RetentionPeriodInDays: 7       # How long execution state is retained

  DurableFunctionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal: { Service: lambda.amazonaws.com }
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: DurableExecutionPolicy
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action:
                  - lambda:CheckpointDurableExecution
                  - lambda:GetDurableExecutionState
                Resource: !GetAtt OrderWorkflow.Arn
```

> **Note:** `RetentionPeriodInDays` (not `RetentionPeriod`) is the correct SAM property name. Scope the durable execution IAM policy to the specific function ARN — avoid `function:*` wildcards.

### Requirements and Limits

- **Runtimes:** Python 3.14+, Node.js 22+
- **SAM CLI:** v1.150.0+ required
- **Max execution duration:** Configured via `ExecutionTimeout` (up to 1 year)
- **Pricing:** Billed only for active compute time — suspension periods do not incur charges

### AI Workflow Patterns

| Pattern | Description |
| --------- | ------------- |
| **Prompt chaining** | Sequential LLM calls where output of one feeds the next; checkpoints prevent re-running successful expensive calls |
| **Human review** | Suspend execution for days/weeks awaiting approval — no compute cost during the wait |
| **LLM as judge** | Parallel model invocations (`context.map`), followed by a comparative evaluation step |
| **Agent with tools** | Agentic loop with tool execution in child contexts and optional human-input suspension |
| **Structured output** | JSON extraction with schema validation; automatic retry on parse failure stays within the same execution |
| **Parallel invocation** | Multiple concurrent LLM calls with independent per-item checkpointing |

---

## AWS Step Functions

Step Functions provides visual workflow orchestration with native integrations to 9,000+ API actions across 200+ AWS services. Define workflows as state machines in Amazon States Language (ASL).

### Standard vs Express Workflows

| | Standard | Express |
| - | ---------- | --------- |
| **Max duration** | 1 year | 5 minutes |
| **Execution semantics** | Exactly-once | At-least-once (async) / At-most-once (sync) |
| **Execution history** | Retained 90 days, queryable via API | CloudWatch Logs only |
| **Max throughput** | 2,000 exec/sec | 100,000 exec/sec |
| **Pricing model** | Per state transition | Per execution count + duration |
| **`.sync` / `.waitForTaskToken`** | Supported | Not supported |
| **Best for** | Auditable, non-idempotent operations | High-volume, idempotent event processing |

**Choose Standard** for: payment processing, order fulfillment, compliance workflows, anything that must never execute twice.

**Choose Express** for: IoT data ingestion, streaming transformations, mobile backends, high-throughput short-lived processing.

### Key State Types

| State | Purpose |
| ------- | --------- |
| `Task` | Execute work — invoke Lambda, call any AWS service via SDK integration |
| `Choice` | Branch based on input data conditions (no `Next` required on branches) |
| `Parallel` | Execute multiple branches concurrently; waits for all branches to complete |
| `Map` | Iterate over an array; use Distributed Map mode for up to 10M items from S3/DynamoDB |
| `Wait` | Pause for a fixed duration or until a specific timestamp |
| `Pass` | Pass input to output, optionally injecting or transforming data |
| `Succeed` / `Fail` | End execution successfully or with an error and cause |

### SAM Template

```yaml
Resources:
  MyWorkflow:
    Type: AWS::Serverless::StateMachine
    Properties:
      DefinitionUri: statemachine/my_workflow.asl.json
      Type: STANDARD                          # or EXPRESS
      DefinitionSubstitutions:
        ProcessFunctionArn: !GetAtt ProcessFunction.Arn
        ResultsTable: !Ref ResultsTable
      Policies:
        - LambdaInvokePolicy:
            FunctionName: !Ref ProcessFunction
        - DynamoDBWritePolicy:
            TableName: !Ref ResultsTable
      Tracing:
        Enabled: true
      Logging:
        Destinations:
          - CloudWatchLogsLogGroup:
              LogGroupArn: !GetAtt WorkflowLogGroup.Arn
        IncludeExecutionData: true
        Level: ERROR                          # Use ALL for debugging, ERROR in production

  WorkflowLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      RetentionInDays: 30
```

### State Machine Definition (ASL)

Use `DefinitionSubstitutions` to inject ARNs — never hardcode them:

```json
{
  "Comment": "Order processing workflow",
  "QueryLanguage": "JSONata",
  "StartAt": "ProcessOrder",
  "States": {
    "ProcessOrder": {
      "Type": "Task",
      "Resource": "${ProcessFunctionArn}",
      "Retry": [
        {
          "ErrorEquals": ["Lambda.ServiceException", "Lambda.AWSLambdaException", "Lambda.TooManyRequestsException"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "Catch": [{ "ErrorEquals": ["States.ALL"], "Next": "HandleError" }],
      "Next": "SaveResult"
    },
    "SaveResult": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:putItem",
      "Arguments": {
        "TableName": "${ResultsTable}",
        "Item": {
          "id": { "S": "{% $states.input.orderId %}" },
          "status": { "S": "completed" }
        }
      },
      "End": true
    },
    "HandleError": {
      "Type": "Fail",
      "Error": "OrderProcessingFailed"
    }
  }
}
```

### JSONata — Recommended Query Language

JSONata is the modern, preferred way to reference and transform data in ASL. It replaces the five JSONPath I/O fields (`InputPath`, `Parameters`, `ResultSelector`, `ResultPath`, `OutputPath`) with just two: `Arguments` (inputs) and `Output` (result shape).

**Enable at the top level** to apply to all states:

```json
{ "QueryLanguage": "JSONata", "StartAt": "...", "States": {...} }
```

**Or per-state** to migrate incrementally:

```json
{ "Type": "Task", "QueryLanguage": "JSONata", ... }
```

**Expression syntax** — wrap expressions in `{% %}`:

```json
"Arguments": {
  "userId": "{% $states.input.user.id %}",
  "greeting": "{% 'Hello, ' & $states.input.user.name %}",
  "total": "{% $sum($states.input.items.price) %}"
}
```

**Built-in Step Functions JSONata functions:**

- `$uuid()` — generate a v4 UUID
- `$parse(str)` — deserialize a JSON string to an object
- `$partition(array, size)` — split array into chunks
- `$range(start, end, step)` — generate a number array
- `$hash(value, algorithm)` — compute MD5/SHA-256/etc. hash

**JSONPath is still supported** and is the default if `QueryLanguage` is omitted — existing state machines do not need to be migrated.

### Integration Patterns

| Pattern | ARN suffix | Behaviour |
| --------- | ----------- | ----------- |
| **Request Response** | *(none)* | Call service, proceed after HTTP 200 |
| **Run a Job** | `.sync` | Call service, wait for job completion |
| **Wait for Callback** | `.waitForTaskToken` | Pass `$$.Task.Token`, pause until `SendTaskSuccess`/`SendTaskFailure` |

**Wait for Callback** is the human-in-the-loop pattern: pass the task token to an external system (email, Slack, ticketing), call `sfn:SendTaskSuccess` with the token when approved.

### SDK Integrations — Avoid Lambda for Simple AWS Calls

Step Functions can call any AWS service API directly without a Lambda intermediary. This saves both cost and latency for simple operations:

```json
"SaveToDynamoDB": {
  "Type": "Task",
  "Resource": "arn:aws:states:::dynamodb:putItem",
  "Arguments": {
    "TableName": "my-table",
    "Item": { "id": { "S": "{% $states.input.id %}" } }
  },
  "End": true
}
```

```json
"PublishEvent": {
  "Type": "Task",
  "Resource": "arn:aws:states:::events:putEvents",
  "Arguments": {
    "Entries": [{
      "EventBusName": "my-bus",
      "Source": "my.service",
      "DetailType": "OrderPlaced",
      "Detail": "{% $states.input %}"
    }]
  },
  "End": true
}
```

Avoiding Lambda intermediaries for simple DynamoDB reads/writes, SNS publishes, SQS sends, and EventBridge puts eliminates invocation latency and cost.

### Distributed Map — Large-Scale Processing

`Map` state with `Mode: DISTRIBUTED` processes up to 10 million items from S3, DynamoDB, or inline arrays, with each item running as an independent child workflow:

```json
"ProcessFiles": {
  "Type": "Map",
  "ItemProcessor": {
    "ProcessorConfig": { "Mode": "DISTRIBUTED", "ExecutionType": "EXPRESS" },
    "StartAt": "ProcessSingleFile",
    "States": { "ProcessSingleFile": { "Type": "Task", "Resource": "${ProcessFunctionArn}", "End": true } }
  },
  "MaxConcurrency": 100,
  "ItemReader": {
    "Resource": "arn:aws:states:::s3:listObjectsV2",
    "Parameters": { "Bucket.$": "$.bucket", "Prefix.$": "$.prefix" }
  },
  "End": true
}
```

### Local Testing

**Recommended: TestState API** — test individual states against real AWS without deploying the full state machine:

```bash
aws stepfunctions test-state \
  --definition '{"Type":"Task","Resource":"${FunctionArn}","End":true}' \
  --role-arn arn:aws:iam::123456789012:role/MyRole \
  --input '{"key":"value"}'
```

**Step Functions Local (Docker)** — run a local emulator for integration testing. Note it is unsupported and does not have full feature parity:

```bash
docker run -p 8083:8083 amazon/aws-stepfunctions-local

# Run alongside sam local start-lambda for Lambda-integrated tests
sam local start-lambda &
docker run -p 8083:8083 \
  -e LAMBDA_ENDPOINT=http://host.docker.internal:3001 \
  amazon/aws-stepfunctions-local
```

Then use the AWS CLI with `--endpoint-url http://localhost:8083` to create and execute state machines locally.

### Anti-Polling Pattern

The typical polling loop — `Wait → Check Status → Choice → loop` — is an expensive anti-pattern in Standard workflows because every state transition is billed. Replace it with the **callback + event-driven** approach:

1. Lambda starts the long-running task and receives a task token (`$$.Task.Token`)
2. Store the task token alongside the job ID in DynamoDB
3. Use `.waitForTaskToken` to pause the state machine at zero cost
4. When the job completes, an EventBridge rule triggers a Lambda that looks up the token and calls `sfn:SendTaskSuccess`

```json
"StartJob": {
  "Type": "Task",
  "Resource": "arn:aws:states:::lambda:invoke.waitForTaskToken",
  "Arguments": {
    "FunctionName": "${StartJobFunctionArn}",
    "Payload": {
      "taskToken": "{% $$.Task.Token %}",
      "input": "{% $states.input %}"
    }
  },
  "HeartbeatSeconds": 3600,
  "Next": "ProcessResult"
}
```

For third-party APIs that don't emit events, pass a callback URL to the external service so it can POST back to your endpoint when done, which then calls `SendTaskSuccess`.

**Lambda Durable Functions alternative:** `context.wait_for_callback()` / `context.waitForCallback()` implements the same pattern without manual token management.

### Fan-Out / Fan-In

| Scale | Recommended approach |
| ------- | --------------------- |
| Up to 40 items | Step Functions `Map` state (Inline mode) |
| Up to 10 million items | Step Functions `Map` state (Distributed mode, child Express workflows) |
| Millions of items, cost-sensitive | Custom: S3 → Lambda fan-out → SQS workers → DynamoDB tracking → aggregation |

For most teams, Step Functions Distributed Map is the right trade-off between cost and operational simplicity. A custom S3+SQS+DynamoDB solution is meaningfully cheaper at very high item counts but carries significant implementation overhead.

### Timeout Handling

Always set **both** `TimeoutSeconds` and `HeartbeatSeconds` on Task states. Without them, a hung downstream call can hold the execution open indefinitely:

```json
"CallExternalAPI": {
  "Type": "Task",
  "Resource": "${FunctionArn}",
  "TimeoutSeconds": 300,
  "HeartbeatSeconds": 60,
  "Retry": [...]
}
```

- `TimeoutSeconds` — maximum total time for the state (including retries)
- `HeartbeatSeconds` — maximum time between heartbeat signals; fails faster when a worker disappears silently

**Handling Express workflow timeouts:** Express workflows do not publish `TIMED_OUT` events to EventBridge. Wrap Express workflows inside a parent Standard workflow — the Standard workflow can catch the timeout and trigger remediation.

### Best Practices

- **Always add `Retry` on Task states** — Lambda returns transient errors (`Lambda.ServiceException`, `Lambda.AWSLambdaException`, `Lambda.TooManyRequestsException`) under load; without retry, these fail the execution
- **Use `Catch` for error routing** — route failures to a dedicated error-handling state rather than letting the execution fail silently
- **Use `DefinitionSubstitutions`** — never hardcode ARNs or table names in `.asl.json` files
- **Use JSONata for new workflows** — it produces simpler, more readable definitions than JSONPath
- **Use SDK integrations directly** — call DynamoDB, SNS, SQS, EventBridge, etc. without a Lambda wrapper for simple operations
- **Enable X-Ray tracing** (`Tracing.Enabled: true`) for end-to-end visibility across Step Functions and Lambda spans
- **Set logging to `Level: ERROR` in production** and `Level: ALL` when debugging; `IncludeExecutionData: true` is required to see input/output in logs
- **Standard workflows**: prefer for non-idempotent operations — exactly-once semantics prevent accidental double-charges or duplicate records
- **Express workflows**: ensure downstream operations are idempotent — at-least-once delivery means tasks may run more than once
