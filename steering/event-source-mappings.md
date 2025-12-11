# Event Source Mappings Guide

## Overview
Event Source Mappings (ESMs) connect AWS services to Lambda functions, enabling event-driven architectures. This guide covers setup, optimization, and troubleshooting for all supported event sources.

## Supported Event Sources

### DynamoDB Streams
**Use Case**: React to data changes in DynamoDB tables
```yaml
DynamoDBEventSourceMapping:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt MyTable.StreamArn
    FunctionName: !Ref MyFunction
    StartingPosition: LATEST
    BatchSize: 10
    ParallelizationFactor: 2
    MaximumBatchingWindowInSeconds: 5
    BisectBatchOnFunctionError: true
    MaximumRetryAttempts: 3
```

**Best Practices**:
- Use `LATEST` for new records only, `TRIM_HORIZON` for all records
- Set appropriate batch size (1-1000) based on record size
- Enable `BisectBatchOnFunctionError` for poison record handling

### Kinesis Streams
**Use Case**: Process real-time streaming data
```yaml
KinesisEventSourceMapping:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt MyStream.Arn
    FunctionName: !Ref MyFunction
    StartingPosition: LATEST
    BatchSize: 100
    ParallelizationFactor: 10
    MaximumBatchingWindowInSeconds: 10
    TumblingWindowInSeconds: 60
```

**Optimization Guidelines**:
- ParallelizationFactor should not exceed shard count
- Higher batch sizes reduce invocation costs
- Use tumbling windows for aggregation scenarios

### SQS Queues
**Use Case**: Decouple application components with reliable messaging
```yaml
SQSEventSourceMapping:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt MyQueue.Arn
    FunctionName: !Ref MyFunction
    BatchSize: 10
    MaximumBatchingWindowInSeconds: 20
    ScalingConfig:
      MaximumConcurrency: 100
    FunctionResponseTypes:
      - ReportBatchItemFailures
```

**FIFO Queue Considerations**:
- BatchSize must be 1 for strict ordering
- MaximumConcurrency should be limited
- Use message group IDs for parallel processing within groups

### MSK/Kafka
**Use Case**: Process high-throughput streaming data from Kafka
```yaml
MSKEventSourceMapping:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !Ref MyMSKCluster
    FunctionName: !Ref MyFunction
    Topics:
      - my-topic
    StartingPosition: LATEST
    BatchSize: 500
    MaximumBatchingWindowInSeconds: 10
    ConsumerGroupId: my-consumer-group
```

**Network Configuration**:
- Ensure Lambda has VPC access to MSK cluster
- Configure security groups for port 9092/9094
- Use IAM authentication or SASL/SCRAM

## Performance Optimization

### Batch Size Optimization
- **Small batches (1-10)**: Lower latency, higher cost
- **Medium batches (10-100)**: Balanced performance
- **Large batches (100-1000)**: Higher throughput, potential timeout risk

### Concurrency Management
```yaml
# Reserved concurrency for predictable workloads
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    ReservedConcurrencyLimit: 50

# Provisioned concurrency for consistent performance
MyFunctionVersion:
  Type: AWS::Lambda::Version
  Properties:
    FunctionName: !Ref MyFunction
    ProvisionedConcurrencyConfig:
      ProvisionedConcurrencyLimit: 10
```

### Error Handling Strategies
```yaml
# Dead letter queue for failed messages
DeadLetterQueue:
  Type: AWS::SQS::Queue
  Properties:
    MessageRetentionPeriod: 1209600  # 14 days

MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    DeadLetterQueue:
      Type: SQS
      TargetArn: !GetAtt DeadLetterQueue.Arn
```

## Monitoring and Observability

### Key Metrics to Monitor
- **Invocation Count**: Total function executions
- **Error Rate**: Percentage of failed invocations
- **Duration**: Function execution time
- **Throttles**: Concurrency limit hits
- **Iterator Age**: Lag in stream processing

### CloudWatch Alarms
```yaml
HighErrorRateAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: !Sub "${FunctionName}-high-error-rate"
    MetricName: Errors
    Namespace: AWS/Lambda
    Statistic: Sum
    Period: 300
    EvaluationPeriods: 2
    Threshold: 10
    ComparisonOperator: GreaterThanThreshold

IteratorAgeAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: !Sub "${FunctionName}-high-iterator-age"
    MetricName: IteratorAge
    Namespace: AWS/Lambda
    Statistic: Maximum
    Period: 300
    Threshold: 60000  # 1 minute in milliseconds
```

## Troubleshooting Common Issues

### High Iterator Age
**Symptoms**: Increasing lag in stream processing
**Solutions**:
- Increase ParallelizationFactor
- Optimize function performance
- Scale up Lambda memory/CPU
- Check for poison records

### Throttling Issues
**Symptoms**: Functions not processing all available records
**Solutions**:
- Increase reserved concurrency
- Optimize function duration
- Implement exponential backoff
- Use SQS for buffering

### Permission Errors
**Common IAM Policies**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kinesis:DescribeStream",
        "kinesis:GetShardIterator",
        "kinesis:GetRecords",
        "kinesis:ListStreams"
      ],
      "Resource": "arn:aws:kinesis:region:account:stream/stream-name"
    }
  ]
}
```

### Network Connectivity (VPC)
**Requirements for VPC-enabled functions**:
- NAT Gateway or VPC endpoints for AWS service access
- Security group rules for outbound traffic
- Subnet routing to internet gateway (for public APIs)

## Event Schema Handling

### Type-Safe Event Processing
```python
from typing import Dict, List, Any
from aws_lambda_powertools.utilities.typing import LambdaContext

def lambda_handler(event: Dict[str, Any], context: LambdaContext) -> Dict[str, Any]:
    # DynamoDB Stream event
    for record in event.get('Records', []):
        if record['eventSource'] == 'aws:dynamodb':
            process_dynamodb_record(record)
        elif record['eventSource'] == 'aws:kinesis':
            process_kinesis_record(record)
    
    return {"statusCode": 200}
```

### EventBridge Schema Registry
Use the schema tools to get type-safe event handling:
1. Search for event schemas: `search_schema`
2. Get schema definition: `describe_schema`
3. Generate typed handlers based on schema

## Cost Optimization

### Right-Sizing Strategies
- Monitor memory utilization in CloudWatch
- Use AWS Lambda Power Tuning for optimal memory
- Implement efficient batch processing
- Consider provisioned concurrency for consistent workloads

### Batch Processing Efficiency
```python
def process_batch_efficiently(records: List[Dict]) -> None:
    # Batch database operations
    with db_connection.batch_writer() as batch:
        for record in records:
            batch.put_item(Item=transform_record(record))
    
    # Use connection pooling
    # Minimize cold start impact
```

This guide provides comprehensive coverage of ESM patterns, optimization strategies, and troubleshooting approaches for building robust event-driven serverless applications.
