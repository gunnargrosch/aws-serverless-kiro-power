# Serverless Optimization Guide

## Memory and CPU Right-Sizing

Lambda allocates CPU proportionally to memory. The goal is to find the configuration where cost-per-invocation is minimized while meeting latency requirements.

**Strategy:**
1. Use `get_metrics` to measure current duration, memory utilization, and invocation count
2. Test with different memory settings using AWS Lambda Power Tuning
3. Choose the memory level where cost (duration x memory price) is lowest

**General guidelines:**
- 128 MB: Lightweight tasks (routing, simple transformations)
- 512 MB: Standard API handlers, moderate data processing
- 1024 MB: Compute-intensive tasks, image processing
- 3008+ MB: ML inference, large data processing

## Cold Start Optimization

Cold starts affect latency on the first invocation after idle time or scaling events.

**Checklist:**
- [ ] Initialize SDK clients and database connections outside the handler function
- [ ] Use `lru_cache` or module-level variables for configuration that doesn't change
- [ ] Minimize deployment package size (exclude dev dependencies, use layers for shared code)
- [ ] Choose a fast-starting runtime (Python, Node.js) for latency-sensitive paths
- [ ] Consider `arm64` architecture for faster cold starts
- [ ] Use provisioned concurrency only for consistently latency-sensitive endpoints

**When to use provisioned concurrency:**
- API endpoints with strict latency SLAs
- Functions called synchronously where cold starts are user-visible
- Not recommended for asynchronous or batch processing workloads

## Cost Optimization

### Decision Framework

| Scenario | Recommendation |
|----------|----------------|
| Unpredictable traffic | On-demand billing, no provisioned concurrency |
| Steady baseline + spikes | Provisioned concurrency for baseline, on-demand for spikes |
| Batch processing | Maximize batch size, optimize memory for cost |
| Infrequently called | Minimize memory, accept cold starts |

### Key Cost Levers
- **Memory**: Lower memory is cheaper per-ms, but if it makes duration longer, net cost may increase
- **Timeout**: Set to actual max expected duration + buffer, not the maximum 900s
- **Reserved concurrency**: Caps maximum concurrent executions to prevent runaway costs
- **Storage**: Use S3 lifecycle policies to transition objects to cheaper tiers
- **Logs**: Set CloudWatch log retention to the minimum needed (7-30 days for dev, longer for prod/compliance)

## API Gateway Optimization

- Enable caching for read-heavy GET endpoints (0.5 GB cache is the minimum size)
- Use request validation at the gateway level to reject bad requests before invoking Lambda
- Use HTTP APIs (v2) instead of REST APIs when you don't need REST API-specific features (cheaper, lower latency)

## DynamoDB Optimization

- Use single-table design with composite keys (PK/SK) for efficient access patterns
- Use `Query` instead of `Scan` wherever possible
- Project only needed attributes to reduce read capacity usage
- Use ON_DEMAND billing for unpredictable workloads, PROVISIONED with auto-scaling for steady workloads
- Use GSIs with KEYS_ONLY projection when you only need to look up primary keys

## Event Source Mapping Tuning

Use `esm_optimize` to get source-specific recommendations. General guidelines:

| Source | Key Tuning Parameters |
|--------|----------------------|
| DynamoDB Streams | `BatchSize` (1-1000), `ParallelizationFactor` (1-10) |
| Kinesis | `BatchSize` (1-10000), `ParallelizationFactor` (1-10), `TumblingWindowInSeconds` |
| SQS | `BatchSize` (1-10000), `MaximumConcurrency`, `MaximumBatchingWindowInSeconds` |
| Kafka/MSK | `BatchSize` (1-10000), `MaximumBatchingWindowInSeconds` |

## Monitoring Checklist

Set up monitoring for these key metrics:

- [ ] **Invocation errors**: Alarm on error rate exceeding threshold
- [ ] **Duration (p99)**: Track latency outliers, not just averages
- [ ] **Throttles**: Alarm on any throttling events
- [ ] **Iterator age**: Alarm when stream processing falls behind (> 60s)
- [ ] **Concurrent executions**: Track against account limits
- [ ] **Dead letter queue depth**: Alarm when messages accumulate in DLQ

Use `get_metrics` to retrieve current values. Enable X-Ray tracing (`Tracing: Active` in SAM template) to identify which downstream calls contribute most to latency.

## Structured Logging

Use structured JSON logging for queryable CloudWatch Logs Insights analysis:

- Include `request_id`, `function_name`, and a `level` field in every log entry
- Log business-relevant context (user ID, operation type) as fields, not in message strings
- Use AWS Lambda Powertools for standardized structured logging
- Set `LOG_LEVEL` via environment variable for per-environment control
