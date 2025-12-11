# Serverless Optimization Guide

## Performance Optimization

### Lambda Function Optimization

#### Memory and CPU Allocation
```yaml
# Right-sizing based on workload
LightweightFunction:
  Type: AWS::Serverless::Function
  Properties:
    MemorySize: 128  # CPU scales proportionally
    Timeout: 15

DataProcessingFunction:
  Type: AWS::Serverless::Function
  Properties:
    MemorySize: 1024  # More CPU for compute-intensive tasks
    Timeout: 300

MLInferenceFunction:
  Type: AWS::Serverless::Function
  Properties:
    MemorySize: 3008  # Maximum memory for ML workloads
    Timeout: 900
```

#### Cold Start Optimization
```python
# Connection pooling outside handler
import boto3
import json
from functools import lru_cache

# Initialize outside handler (reused across invocations)
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('MyTable')

@lru_cache(maxsize=128)
def get_config(key):
    """Cache configuration values"""
    return table.get_item(Key={'config_key': key})['Item']['value']

def lambda_handler(event, context):
    # Handler logic here
    config_value = get_config('api_endpoint')
    return process_request(event, config_value)
```

#### Package Size Optimization
```dockerfile
# Multi-stage build for smaller packages
FROM public.ecr.aws/lambda/python:3.12 as builder
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt -t /opt/python

FROM public.ecr.aws/lambda/python:3.12
COPY --from=builder /opt/python ${LAMBDA_RUNTIME_DIR}
COPY app.py ${LAMBDA_TASK_ROOT}
CMD ["app.lambda_handler"]
```

### API Gateway Optimization

#### Caching Configuration
```yaml
ApiGatewayApi:
  Type: AWS::Serverless::Api
  Properties:
    CacheClusterEnabled: true
    CacheClusterSize: 0.5
    MethodSettings:
      - ResourcePath: "/users/*"
        HttpMethod: GET
        CachingEnabled: true
        CacheTtlInSeconds: 300
        CacheKeyParameters:
          - method.request.path.userId
```

#### Request Validation
```yaml
RequestValidator:
  Type: AWS::ApiGateway::RequestValidator
  Properties:
    RestApiId: !Ref ApiGatewayApi
    ValidateRequestBody: true
    ValidateRequestParameters: true

MyApiMethod:
  Type: AWS::ApiGateway::Method
  Properties:
    RequestValidatorId: !Ref RequestValidator
    RequestModels:
      application/json: !Ref UserModel
```

### DynamoDB Optimization

#### Table Design Best Practices
```yaml
OptimizedTable:
  Type: AWS::DynamoDB::Table
  Properties:
    BillingMode: ON_DEMAND  # Auto-scaling for unpredictable workloads
    AttributeDefinitions:
      - AttributeName: PK
        AttributeType: S
      - AttributeName: SK
        AttributeType: S
      - AttributeName: GSI1PK
        AttributeType: S
    KeySchema:
      - AttributeName: PK
        KeyType: HASH
      - AttributeName: SK
        KeyType: RANGE
    GlobalSecondaryIndexes:
      - IndexName: GSI1
        KeySchema:
          - AttributeName: GSI1PK
            KeyType: HASH
          - AttributeName: SK
            KeyType: RANGE
        Projection:
          ProjectionType: KEYS_ONLY
```

#### Query Optimization
```python
# Efficient query patterns
def get_user_posts(user_id, limit=20):
    """Use query instead of scan for better performance"""
    response = table.query(
        IndexName='GSI1',
        KeyConditionExpression=Key('GSI1PK').eq(f'USER#{user_id}'),
        ScanIndexForward=False,  # Sort descending
        Limit=limit
    )
    return response['Items']

def batch_get_items(item_keys):
    """Batch operations for efficiency"""
    response = dynamodb.batch_get_item(
        RequestItems={
            'MyTable': {
                'Keys': item_keys
            }
        }
    )
    return response['Responses']['MyTable']
```

## Cost Optimization

### Lambda Cost Management

#### Provisioned Concurrency Strategy
```yaml
# Use for predictable, latency-sensitive workloads
ProvisionedConcurrency:
  Type: AWS::Lambda::ProvisionedConcurrencyConfig
  Properties:
    FunctionName: !Ref CriticalFunction
    Qualifier: !GetAtt FunctionVersion.Version
    ProvisionedConcurrencyLimit: 10

# Schedule-based provisioning
ProvisioningSchedule:
  Type: AWS::Events::Rule
  Properties:
    ScheduleExpression: "cron(0 8 * * MON-FRI)"  # Business hours
    Targets:
      - Arn: !Sub "${ScalingFunction.Arn}"
        Id: "ScaleUpTarget"
```

#### Reserved Concurrency for Cost Control
```yaml
# Prevent runaway costs
CostControlledFunction:
  Type: AWS::Serverless::Function
  Properties:
    ReservedConcurrencyLimit: 50  # Maximum concurrent executions
    Environment:
      Variables:
        MAX_BATCH_SIZE: 100
```

### Storage Cost Optimization

#### S3 Lifecycle Policies
```yaml
S3Bucket:
  Type: AWS::S3::Bucket
  Properties:
    LifecycleConfiguration:
      Rules:
        - Id: OptimizeStorage
          Status: Enabled
          Transitions:
            - TransitionInDays: 30
              StorageClass: STANDARD_IA
            - TransitionInDays: 90
              StorageClass: GLACIER
            - TransitionInDays: 365
              StorageClass: DEEP_ARCHIVE
          ExpirationInDays: 2555  # 7 years
```

#### CloudWatch Logs Retention
```yaml
LogGroup:
  Type: AWS::Logs::LogGroup
  Properties:
    LogGroupName: !Sub "/aws/lambda/${FunctionName}"
    RetentionInDays: 7  # Adjust based on compliance needs
```

## Scalability Optimization

### Event Source Mapping Tuning

#### Kinesis Stream Optimization
```yaml
KinesisESM:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt KinesisStream.Arn
    FunctionName: !Ref ProcessorFunction
    BatchSize: 500  # Larger batches for throughput
    ParallelizationFactor: 10  # Match shard count
    MaximumBatchingWindowInSeconds: 10
    StartingPosition: LATEST
    TumblingWindowInSeconds: 60  # For aggregation
```

#### SQS Scaling Configuration
```yaml
SQSEventSourceMapping:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt ProcessingQueue.Arn
    FunctionName: !Ref ProcessorFunction
    BatchSize: 10
    ScalingConfig:
      MaximumConcurrency: 1000  # High throughput
    FunctionResponseTypes:
      - ReportBatchItemFailures  # Partial batch failure handling
```

### Auto Scaling Patterns

#### Application Auto Scaling
```yaml
ScalableTarget:
  Type: AWS::ApplicationAutoScaling::ScalableTarget
  Properties:
    ServiceNamespace: dynamodb
    ResourceId: table/MyTable
    ScalableDimension: dynamodb:table:ReadCapacityUnits
    MinCapacity: 5
    MaxCapacity: 1000

ScalingPolicy:
  Type: AWS::ApplicationAutoScaling::ScalingPolicy
  Properties:
    PolicyName: ReadCapacityScalingPolicy
    PolicyType: TargetTrackingScaling
    ScalingTargetId: !Ref ScalableTarget
    TargetTrackingScalingPolicyConfiguration:
      TargetValue: 70.0
      PredefinedMetricSpecification:
        PredefinedMetricType: DynamoDBReadCapacityUtilization
```

## Monitoring and Observability

### Custom Metrics and Alarms
```yaml
BusinessMetricFilter:
  Type: AWS::Logs::MetricFilter
  Properties:
    LogGroupName: !Ref LogGroup
    FilterPattern: "[timestamp, requestId, level=\"ERROR\", message]"
    MetricTransformations:
      - MetricNamespace: MyApp/Errors
        MetricName: ApplicationErrors
        MetricValue: "1"

HighErrorRateAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: !Sub "${AWS::StackName}-high-error-rate"
    MetricName: ApplicationErrors
    Namespace: MyApp/Errors
    Statistic: Sum
    Period: 300
    EvaluationPeriods: 2
    Threshold: 10
    ComparisonOperator: GreaterThanThreshold
    AlarmActions:
      - !Ref SNSAlert
```

### X-Ray Tracing Optimization
```yaml
TracedFunction:
  Type: AWS::Serverless::Function
  Properties:
    Tracing: Active
    Environment:
      Variables:
        _X_AMZN_TRACE_ID: !Ref AWS::NoValue
        AWS_XRAY_TRACING_NAME: MyService
        AWS_XRAY_CONTEXT_MISSING: LOG_ERROR
```

```python
# Custom X-Ray segments
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Patch AWS SDK calls
patch_all()

@xray_recorder.capture('database_operation')
def query_database(query):
    # Database operation
    return result

def lambda_handler(event, context):
    with xray_recorder.in_subsegment('business_logic'):
        result = process_business_logic(event)
    return result
```

## Security Optimization

### IAM Policy Optimization
```yaml
# Least privilege principle
OptimizedExecutionRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
          Condition:
            StringEquals:
              'aws:SourceAccount': !Ref AWS::AccountId
    Policies:
      - PolicyName: MinimalDynamoDBAccess
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - dynamodb:GetItem
                - dynamodb:PutItem
              Resource: !GetAtt MyTable.Arn
              Condition:
                ForAllValues:StringEquals:
                  'dynamodb:Attributes': ['PK', 'SK', 'data']
```

### VPC Optimization
```yaml
# VPC endpoints for AWS services (avoid NAT Gateway costs)
DynamoDBEndpoint:
  Type: AWS::EC2::VPCEndpoint
  Properties:
    VpcId: !Ref VPC
    ServiceName: !Sub "com.amazonaws.${AWS::Region}.dynamodb"
    VpcEndpointType: Gateway
    RouteTableIds:
      - !Ref PrivateRouteTable

S3Endpoint:
  Type: AWS::EC2::VPCEndpoint
  Properties:
    VpcId: !Ref VPC
    ServiceName: !Sub "com.amazonaws.${AWS::Region}.s3"
    VpcEndpointType: Gateway
```

## Development Workflow Optimization

### Local Development Setup
```yaml
# docker-compose.yml for local development
version: '3.8'
services:
  dynamodb-local:
    image: amazon/dynamodb-local
    ports:
      - "8000:8000"
    command: ["-jar", "DynamoDBLocal.jar", "-sharedDb", "-inMemory"]
  
  localstack:
    image: localstack/localstack
    ports:
      - "4566:4566"
    environment:
      - SERVICES=s3,sqs,sns
      - DEBUG=1
```

### Testing Strategy
```python
# Unit tests with mocking
import boto3
import pytest
from moto import mock_dynamodb
from myapp import lambda_handler

@mock_dynamodb
def test_lambda_handler():
    # Setup mock DynamoDB
    dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
    table = dynamodb.create_table(
        TableName='test-table',
        KeySchema=[{'AttributeName': 'id', 'KeyType': 'HASH'}],
        AttributeDefinitions=[{'AttributeName': 'id', 'AttributeType': 'S'}],
        BillingMode='PAY_PER_REQUEST'
    )
    
    # Test function
    event = {'id': 'test-123'}
    result = lambda_handler(event, {})
    
    assert result['statusCode'] == 200
```

### CI/CD Optimization
```yaml
# Optimized build pipeline
name: Deploy Serverless App
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Tests
        run: |
          pip install -r requirements-dev.txt
          pytest tests/ --cov=src/
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy with SAM
        run: |
          sam build --use-container --parallel
          sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
```

This optimization guide covers performance, cost, scalability, security, and development workflow improvements for serverless applications on AWS.
