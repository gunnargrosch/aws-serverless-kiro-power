# Event Source Mappings Guide

## Overview

Event Source Mappings (ESMs) connect AWS services to Lambda functions, enabling event-driven architectures. Use `esm_guidance` for setup recommendations and `esm_optimize` for performance tuning.

## Supported Event Sources

### DynamoDB Streams
**Use case:** React to data changes in DynamoDB tables

**Key configuration:**
- `StartingPosition`: `LATEST` for new records only, `TRIM_HORIZON` for all
- `BatchSize`: 1-1000 (default 100)
- `ParallelizationFactor`: 1-10 (default 1, increase for throughput)
- `BisectBatchOnFunctionError`: Enable to isolate poison records
- `MaximumRetryAttempts`: Set to prevent infinite retries (default unlimited)
- `MaximumBatchingWindowInSeconds`: Buffer time before invoking (0-300)

**Best practices:**
- Enable `BisectBatchOnFunctionError` and set `MaximumRetryAttempts` to 3
- Configure a dead-letter queue for records that exhaust retries
- Use `ParallelizationFactor` > 1 when processing can't keep up

### Kinesis Streams
**Use case:** Process real-time streaming data

**Key configuration:**
- `BatchSize`: 1-10000 (default 100)
- `ParallelizationFactor`: 1-10 (should not exceed shard count)
- `MaximumBatchingWindowInSeconds`: Buffer time (0-300)
- `TumblingWindowInSeconds`: For aggregation scenarios (0-900)
- `StartingPosition`: `LATEST` or `TRIM_HORIZON`

**Best practices:**
- Higher batch sizes reduce invocation costs but increase timeout risk
- Use tumbling windows for time-based aggregation (counts, sums, averages)
- Enable enhanced fan-out when multiple consumers read from the same stream

### SQS Queues
**Use case:** Decouple components with reliable messaging

**Key configuration:**
- `BatchSize`: 1-10000 (default 10)
- `MaximumBatchingWindowInSeconds`: Buffer time (0-300)
- `MaximumConcurrency`: Limit concurrent Lambda invocations
- `FunctionResponseTypes`: Set to `["ReportBatchItemFailures"]` to avoid reprocessing successful messages

**FIFO queue considerations:**
- Use `BatchSize: 1` for strict ordering
- Limit `MaximumConcurrency` to prevent out-of-order processing
- Use message group IDs for parallel processing within groups

**Best practices:**
- Always enable `ReportBatchItemFailures` for partial failure handling
- Set queue `VisibilityTimeout` >= Lambda function timeout
- Configure a DLQ with `maxReceiveCount` of 3-5

### MSK/Kafka
**Use case:** Process high-throughput streaming data from Kafka

**Key configuration:**
- `Topics`: List of Kafka topics to consume
- `BatchSize`: 1-10000 (default 100)
- `MaximumBatchingWindowInSeconds`: Buffer time (0-300)
- `StartingPosition`: `LATEST` or `TRIM_HORIZON`
- `ConsumerGroupId`: Consumer group identifier

**Network requirements:**
- Lambda must have VPC access to the MSK cluster
- Security groups must allow traffic on ports 9092 (plaintext) or 9094 (TLS)
- Use IAM authentication or SASL/SCRAM for authentication

**Best practices:**
- Use `esm_kafka_troubleshoot` for connectivity issues
- Generate IAM policies with `secure_esm_msk_policy`

## Batch Size Guidelines

| Priority | Small (1-10) | Medium (10-100) | Large (100-1000+) |
|----------|-------------|-----------------|-------------------|
| **Latency** | Lowest | Moderate | Higher |
| **Cost** | Higher (more invocations) | Balanced | Lower (fewer invocations) |
| **Timeout risk** | Low | Low | Higher (more processing per invocation) |

## Error Handling

- **Stream sources** (DynamoDB, Kinesis): Records retry until success, expiry, or max retries. Enable `BisectBatchOnFunctionError` and set `MaximumRetryAttempts`.
- **SQS**: Failed messages return to the queue after visibility timeout. Use `ReportBatchItemFailures` for partial batch success.
- **Kafka**: Similar to stream sources. Failed batches retry based on ESM configuration.

Always configure a dead-letter queue or on-failure destination to capture records that cannot be processed.

## Monitoring

**Key metrics to alarm on:**
- `IteratorAge` (streams): Lag in processing. Alert when > 60 seconds.
- `Errors` and error rate: Failed invocations from the ESM.
- `Throttles`: Function concurrency limits being hit.
- DLQ message count: Messages that exhausted retries.

## Schema Integration

For type-safe event processing with EventBridge:
1. Use `search_schema` to find event schemas
2. Use `describe_schema` to get the full definition
3. Generate typed handlers based on the schema
