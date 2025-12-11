# Serverless Troubleshooting Guide

## Common Lambda Issues

### Cold Start Problems

#### Symptoms
- High latency on first invocation
- Timeout errors during initialization
- Inconsistent response times

#### Diagnosis
```bash
# Check CloudWatch metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value=MyFunction \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 300 \
  --statistics Average,Maximum
```

#### Solutions
```yaml
# Provisioned concurrency for critical functions
ProvisionedConcurrency:
  Type: AWS::Lambda::ProvisionedConcurrencyConfig
  Properties:
    FunctionName: !Ref CriticalFunction
    Qualifier: !GetAtt FunctionVersion.Version
    ProvisionedConcurrencyLimit: 5

# Optimize package size
OptimizedFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: 
      Bucket: !Ref DeploymentBucket
      Key: optimized-package.zip
    Layers:
      - !Ref SharedDependenciesLayer
```

### Memory and Timeout Issues

#### Out of Memory Errors
```python
# Monitor memory usage
import psutil
import os

def lambda_handler(event, context):
    # Log memory usage
    memory_info = psutil.virtual_memory()
    print(f"Memory usage: {memory_info.percent}%")
    print(f"Available memory: {memory_info.available / 1024 / 1024:.2f} MB")
    
    # Check Lambda memory limit
    lambda_memory = int(os.environ.get('AWS_LAMBDA_FUNCTION_MEMORY_SIZE', '128'))
    print(f"Lambda memory limit: {lambda_memory} MB")
```

#### Timeout Troubleshooting
```yaml
# Increase timeout and memory
TimeoutProneFunction:
  Type: AWS::Serverless::Function
  Properties:
    MemorySize: 1024  # More memory = more CPU
    Timeout: 300      # Increase timeout
    Environment:
      Variables:
        PYTHONUNBUFFERED: "1"  # Immediate log output
```

### Permission Issues

#### IAM Role Debugging
```bash
# Test IAM permissions
aws sts get-caller-identity
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:role/lambda-role \
  --action-names dynamodb:GetItem \
  --resource-arns arn:aws:dynamodb:us-east-1:123456789012:table/MyTable
```

#### Common IAM Policies
```yaml
LambdaExecutionRole:
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
                - !GetAtt MyTable.Arn
                - !Sub "${MyTable.Arn}/index/*"
```

## API Gateway Issues

### CORS Problems

#### Symptoms
- Browser blocking requests
- "Access-Control-Allow-Origin" errors
- OPTIONS requests failing

#### Solutions
```yaml
ApiGatewayApi:
  Type: AWS::Serverless::Api
  Properties:
    Cors:
      AllowMethods: "'GET,POST,PUT,DELETE,OPTIONS'"
      AllowHeaders: "'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'"
      AllowOrigin: "'*'"  # Use specific domains in production
      MaxAge: "'600'"

# Manual CORS for complex scenarios
CorsOptionsMethod:
  Type: AWS::ApiGateway::Method
  Properties:
    RestApiId: !Ref ApiGatewayApi
    ResourceId: !Ref ApiResource
    HttpMethod: OPTIONS
    AuthorizationType: NONE
    Integration:
      Type: MOCK
      IntegrationResponses:
        - StatusCode: 200
          ResponseParameters:
            method.response.header.Access-Control-Allow-Headers: "'Content-Type,X-Amz-Date,Authorization'"
            method.response.header.Access-Control-Allow-Methods: "'GET,POST,PUT,DELETE'"
            method.response.header.Access-Control-Allow-Origin: "'*'"
```

### Request/Response Issues

#### Large Payload Problems
```python
# Handle large payloads with S3
import boto3
import json
import base64

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Check payload size
    payload_size = len(json.dumps(event))
    
    if payload_size > 6 * 1024 * 1024:  # 6MB limit
        # Store in S3 for large payloads
        s3_key = f"large-payloads/{context.aws_request_id}"
        s3.put_object(
            Bucket='my-large-payloads-bucket',
            Key=s3_key,
            Body=json.dumps(event)
        )
        return {
            'statusCode': 200,
            'body': json.dumps({'s3_key': s3_key})
        }
```

#### Request Validation
```yaml
RequestValidator:
  Type: AWS::ApiGateway::RequestValidator
  Properties:
    RestApiId: !Ref ApiGatewayApi
    ValidateRequestBody: true
    ValidateRequestParameters: true

UserModel:
  Type: AWS::ApiGateway::Model
  Properties:
    RestApiId: !Ref ApiGatewayApi
    ContentType: application/json
    Schema:
      type: object
      required: [name, email]
      properties:
        name:
          type: string
          minLength: 1
        email:
          type: string
          format: email
```

## Event Source Mapping Issues

### DynamoDB Streams

#### High Iterator Age
```bash
# Monitor iterator age
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name IteratorAge \
  --dimensions Name=FunctionName,Value=MyStreamProcessor \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Maximum
```

#### Solutions
```yaml
OptimizedDynamoDBESM:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt MyTable.StreamArn
    FunctionName: !Ref ProcessorFunction
    StartingPosition: LATEST
    BatchSize: 100  # Increase for throughput
    ParallelizationFactor: 10  # Increase parallelization
    MaximumBatchingWindowInSeconds: 5
    BisectBatchOnFunctionError: true  # Handle poison records
    MaximumRetryAttempts: 3
    MaximumRecordAgeInSeconds: 3600
```

### Kinesis Streams

#### Shard Throttling
```python
# Monitor shard metrics
def check_shard_utilization():
    cloudwatch = boto3.client('cloudwatch')
    
    response = cloudwatch.get_metric_statistics(
        Namespace='AWS/Kinesis',
        MetricName='IncomingRecords',
        Dimensions=[
            {'Name': 'StreamName', 'Value': 'MyStream'}
        ],
        StartTime=datetime.utcnow() - timedelta(hours=1),
        EndTime=datetime.utcnow(),
        Period=300,
        Statistics=['Sum']
    )
    
    return response['Datapoints']
```

#### Scaling Solutions
```yaml
KinesisStream:
  Type: AWS::Kinesis::Stream
  Properties:
    ShardCount: 10  # Scale based on throughput needs
    StreamModeDetails:
      StreamMode: PROVISIONED
    
# Or use on-demand mode
OnDemandKinesisStream:
  Type: AWS::Kinesis::Stream
  Properties:
    StreamModeDetails:
      StreamMode: ON_DEMAND
```

### SQS Issues

#### Dead Letter Queue Setup
```yaml
ProcessingQueue:
  Type: AWS::SQS::Queue
  Properties:
    VisibilityTimeoutSeconds: 300  # Match Lambda timeout
    MessageRetentionPeriod: 1209600  # 14 days
    RedrivePolicy:
      deadLetterTargetArn: !GetAtt DeadLetterQueue.Arn
      maxReceiveCount: 3

DeadLetterQueue:
  Type: AWS::SQS::Queue
  Properties:
    MessageRetentionPeriod: 1209600  # 14 days for investigation
```

#### FIFO Queue Issues
```yaml
FIFOQueue:
  Type: AWS::SQS::Queue
  Properties:
    QueueName: MyQueue.fifo
    FifoQueue: true
    ContentBasedDeduplication: true
    DeduplicationScope: messageGroup  # Per message group
    FifoThroughputLimit: perMessageGroupId

FIFOEventSourceMapping:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt FIFOQueue.Arn
    FunctionName: !Ref ProcessorFunction
    BatchSize: 1  # Required for strict ordering
    MaximumConcurrency: 2  # Limited for FIFO
```

## VPC and Networking Issues

### VPC Configuration Problems

#### ENI Exhaustion
```yaml
# Monitor ENI usage
ENIAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: Lambda-ENI-Usage-High
    MetricName: ConcurrentExecutions
    Namespace: AWS/Lambda
    Statistic: Maximum
    Period: 300
    EvaluationPeriods: 2
    Threshold: 900  # Adjust based on subnet size
    ComparisonOperator: GreaterThanThreshold
```

#### Solutions
```yaml
# Use multiple subnets
VpcConfig:
  SecurityGroupIds:
    - !Ref LambdaSecurityGroup
  SubnetIds:
    - !Ref PrivateSubnet1
    - !Ref PrivateSubnet2
    - !Ref PrivateSubnet3

# VPC endpoints to avoid NAT Gateway
DynamoDBEndpoint:
  Type: AWS::EC2::VPCEndpoint
  Properties:
    VpcId: !Ref VPC
    ServiceName: !Sub "com.amazonaws.${AWS::Region}.dynamodb"
    VpcEndpointType: Gateway
```

### Security Group Issues

#### Debugging Connectivity
```bash
# Test security group rules
aws ec2 describe-security-groups --group-ids sg-12345678
aws ec2 authorize-security-group-egress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

#### Proper Security Group Configuration
```yaml
LambdaSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Lambda function security group
    VpcId: !Ref VPC
    SecurityGroupEgress:
      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        CidrIp: 0.0.0.0/0  # HTTPS outbound
      - IpProtocol: tcp
        FromPort: 5432
        ToPort: 5432
        SourceSecurityGroupId: !Ref DatabaseSecurityGroup  # Database access
```

## Monitoring and Debugging

### CloudWatch Logs Analysis

#### Log Aggregation Queries
```bash
# CloudWatch Insights queries
aws logs start-query \
  --log-group-name "/aws/lambda/my-function" \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string '
    fields @timestamp, @message
    | filter @message like /ERROR/
    | sort @timestamp desc
    | limit 100
  '
```

#### Custom Log Parsing
```python
import json
import logging

# Structured logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Log structured data
        logger.info(json.dumps({
            'event': 'function_start',
            'request_id': context.aws_request_id,
            'function_name': context.function_name,
            'remaining_time': context.get_remaining_time_in_millis()
        }))
        
        # Your function logic
        result = process_event(event)
        
        logger.info(json.dumps({
            'event': 'function_success',
            'request_id': context.aws_request_id,
            'result_size': len(str(result))
        }))
        
        return result
        
    except Exception as e:
        logger.error(json.dumps({
            'event': 'function_error',
            'request_id': context.aws_request_id,
            'error': str(e),
            'error_type': type(e).__name__
        }))
        raise
```

### X-Ray Tracing

#### Advanced Tracing Setup
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all
import boto3

# Patch AWS SDK calls
patch_all()

@xray_recorder.capture('database_query')
def query_database(table_name, key):
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table(table_name)
    
    # Add custom annotations
    xray_recorder.current_subsegment().put_annotation('table_name', table_name)
    xray_recorder.current_subsegment().put_metadata('query_key', key)
    
    return table.get_item(Key=key)

def lambda_handler(event, context):
    # Custom segment for business logic
    with xray_recorder.in_subsegment('business_logic') as subsegment:
        subsegment.put_annotation('user_id', event.get('user_id'))
        result = process_business_logic(event)
    
    return result
```

### Performance Monitoring

#### Custom Metrics
```python
import boto3
from datetime import datetime

cloudwatch = boto3.client('cloudwatch')

def publish_custom_metric(metric_name, value, unit='Count'):
    cloudwatch.put_metric_data(
        Namespace='MyApp/Performance',
        MetricData=[
            {
                'MetricName': metric_name,
                'Value': value,
                'Unit': unit,
                'Timestamp': datetime.utcnow()
            }
        ]
    )

def lambda_handler(event, context):
    start_time = datetime.utcnow()
    
    try:
        result = process_event(event)
        
        # Publish success metric
        publish_custom_metric('ProcessingSuccess', 1)
        
        return result
        
    except Exception as e:
        # Publish error metric
        publish_custom_metric('ProcessingError', 1)
        raise
        
    finally:
        # Publish processing time
        processing_time = (datetime.utcnow() - start_time).total_seconds()
        publish_custom_metric('ProcessingTime', processing_time, 'Seconds')
```

This troubleshooting guide provides systematic approaches to diagnosing and resolving common serverless application issues on AWS.
