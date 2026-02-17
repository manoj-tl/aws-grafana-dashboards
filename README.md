# AWS Grafana Dashboards

A comprehensive collection of Grafana dashboards for monitoring AWS services using CloudWatch as the data source.

## Overview

This repository contains 9 production-ready Grafana dashboards organized into three categories:

1. **Summary Dashboards** - High-level health overview across all resources
2. **Detailed Dashboards** - Deep dive into individual resource metrics
3. **Flow Dashboards** - End-to-end cross-service views

## Dashboard Structure

```
aws-grafana-dashboards/
├── lambda/
│   ├── lambda-summary.json      # Overall Lambda health
│   └── lambda-detailed.json     # Per-function metrics
├── sqs/
│   ├── sqs-summary.json         # Overall SQS health
│   └── sqs-detailed.json        # Per-queue metrics
├── sns/
│   ├── sns-summary.json         # Overall SNS health
│   └── sns-detailed.json        # Per-topic metrics
├── dynamodb/
│   ├── dynamodb-summary.json    # Overall DynamoDB health
│   └── dynamodb-detailed.json   # Per-table metrics
├── messaging/
│   └── messaging-flow.json      # SNS → SQS → DLQ message flow
└── README.md
```

## Features

- **AWS GovCloud Support** - Configured for us-gov-west-1 (default) and us-gov-east-1
- **CloudWatch Data Source** - Uses native CloudWatch metrics
- **Drill-Down Navigation** - Links between summary and detailed dashboards
- **Best Practice Time Ranges**:
  - Summary dashboards: 6 hours (default)
  - Detailed dashboards: 1 hour (default)
- **Threshold-based Alerting** - Visual thresholds on key metrics

## Dashboards

### AWS Lambda

| Dashboard | UID | Description |
|-----------|-----|-------------|
| Lambda Summary | `lambda-summary` | Total invocations, errors, throttles, duration, concurrent executions, top functions |
| Lambda Detailed | `lambda-detailed` | Per-function invocations, errors, duration percentiles (p50/p90/p99), async events, provisioned concurrency |

**Key Metrics:**
- Invocations, Errors, Error Rate
- Duration (avg, p50, p90, p99)
- Concurrent Executions
- Throttles
- Iterator Age (stream-based)
- Provisioned Concurrency Utilization

### Amazon SQS

| Dashboard | UID | Description |
|-----------|-----|-------------|
| SQS Summary | `sqs-summary` | Messages sent/received/deleted, queue depth, age of oldest message, top queues |
| SQS Detailed | `sqs-detailed` | Per-queue message operations, queue depth (visible/in-flight/delayed), empty receives |

**Key Metrics:**
- NumberOfMessagesSent/Received/Deleted
- ApproximateNumberOfMessagesVisible
- ApproximateNumberOfMessagesNotVisible
- ApproximateAgeOfOldestMessage
- NumberOfEmptyReceives

### Amazon SNS

| Dashboard | UID | Description |
|-----------|-----|-------------|
| SNS Summary | `sns-summary` | Messages published, notifications delivered/failed, delivery success rate, top topics |
| SNS Detailed | `sns-detailed` | Per-topic publishing, delivery status, message size, filtering metrics, SMS metrics |

**Key Metrics:**
- NumberOfMessagesPublished
- NumberOfNotificationsDelivered/Failed
- Delivery Success Rate
- PublishSize
- NumberOfNotificationsFilteredOut
- SMSSuccessRate

### Amazon DynamoDB

| Dashboard | UID | Description |
|-----------|-----|-------------|
| DynamoDB Summary | `dynamodb-summary` | Consumed capacity, throttles, system errors, latency, top tables |
| DynamoDB Detailed | `dynamodb-detailed` | Per-table capacity (consumed vs provisioned), latency percentiles, throttles, errors, replication latency |

**Key Metrics:**
- ConsumedReadCapacityUnits/ConsumedWriteCapacityUnits
- ProvisionedReadCapacityUnits/ProvisionedWriteCapacityUnits
- ReadThrottledRequests/WriteThrottledRequests
- SuccessfulRequestLatency (avg, p90, p99)
- SystemErrors/UserErrors
- ReplicationLatency (Global Tables)

### Messaging Flow (SNS → SQS → DLQ)

| Dashboard | UID | Description |
|-----------|-----|-------------|
| Messaging Flow | `messaging-flow` | End-to-end view: SNS delivery success/failure, SQS queue backlog & delayed messages, DLQ depth & age |

**Key Metrics:**
- SNS: NumberOfMessagesPublished, NumberOfNotificationsDelivered, NumberOfNotificationsFailed, Delivery Success Rate
- SQS: NumberOfMessagesSent/Received/Deleted, ApproximateNumberOfMessagesDelayed, ApproximateNumberOfMessagesVisible
- DLQ: ApproximateNumberOfMessagesVisible, ApproximateAgeOfOldestMessage, NumberOfMessagesSent (new failures arriving)

## Installation

### Prerequisites

1. Grafana 9.0+ installed
2. AWS CloudWatch data source configured in Grafana
3. Appropriate IAM permissions for CloudWatch metrics access

### IAM Policy

Ensure your Grafana CloudWatch data source has the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:DescribeAlarmsForMetric",
        "cloudwatch:DescribeAlarmHistory",
        "cloudwatch:DescribeAlarms",
        "cloudwatch:ListMetrics",
        "cloudwatch:GetMetricData",
        "cloudwatch:GetMetricStatistics"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:DescribeLogGroups",
        "logs:GetLogGroupFields",
        "logs:StartQuery",
        "logs:StopQuery",
        "logs:GetQueryResults",
        "logs:GetLogEvents"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeRegions"
      ],
      "Resource": "*"
    }
  ]
}
```

### Import Dashboards

#### Option 1: Grafana UI

1. Navigate to **Dashboards** > **Import**
2. Upload the JSON file or paste the JSON content
3. Select your CloudWatch data source
4. Click **Import**

#### Option 2: Grafana API

```bash
# Replace with your Grafana URL and API key
GRAFANA_URL="http://localhost:3000"
GRAFANA_API_KEY="your-api-key"

# Import all dashboards
for file in lambda/*.json sqs/*.json sns/*.json dynamodb/*.json; do
  curl -X POST \
    -H "Authorization: Bearer $GRAFANA_API_KEY" \
    -H "Content-Type: application/json" \
    -d @"$file" \
    "$GRAFANA_URL/api/dashboards/db"
done
```

#### Option 3: Grafana Provisioning

Add to your Grafana provisioning configuration:

```yaml
# /etc/grafana/provisioning/dashboards/aws.yaml
apiVersion: 1

providers:
  - name: 'AWS Dashboards'
    orgId: 1
    folder: 'AWS'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 30
    options:
      path: /path/to/aws-grafana-dashboards
```

## Template Variables

All dashboards use the following template variables:

| Variable | Description |
|----------|-------------|
| `datasource` | CloudWatch data source selector |
| `region` | AWS GovCloud region (us-gov-west-1 default, us-gov-east-1) |
| `function` | Lambda function name (detailed dashboard only) |
| `queue` | SQS queue name (detailed dashboard only) |
| `topic` | SNS topic name (detailed dashboard only) |
| `table` | DynamoDB table name (detailed dashboard only) |

## Alerting

Dashboards include visual thresholds that can be converted to alerts:

### Lambda
- Error Rate > 1% (warning), > 5% (critical)
- Duration > 1s (warning), > 5s (critical)
- Throttles > 0 (warning), > 10 (critical)

### SQS
- Messages Visible > 100 (warning), > 1000 (critical)
- Age of Oldest Message > 1min (warning), > 5min (critical)

### SNS
- Delivery Success Rate < 99% (warning), < 95% (critical)
- Notifications Failed > 0 (warning), > 10 (critical)

### DynamoDB
- Throttled Requests > 0 (warning), > 10 (critical)
- System Errors > 0 (warning), > 10 (critical)
- Latency > 10ms (warning), > 50ms (critical)

## Customization

### Modifying Thresholds

Edit the `thresholds` section in each panel:

```json
"thresholds": {
  "mode": "absolute",
  "steps": [
    { "color": "green", "value": null },
    { "color": "yellow", "value": 100 },
    { "color": "red", "value": 1000 }
  ]
}
```

### Adding Filters

To filter by resource name prefix, modify the dimension query:

```json
"dimensions": {
  "FunctionName": "prod-*"
}
```

### Changing Time Range

Modify the `time` section:

```json
"time": {
  "from": "now-24h",
  "to": "now"
}
```

## Best Practices

1. **Start with Summary** - Use summary dashboards for daily monitoring
2. **Drill Down** - Click links to detailed dashboards when investigating issues
3. **Set Alerts** - Convert visual thresholds to Grafana alerts for proactive monitoring
4. **Use Annotations** - Mark deployments and incidents for correlation
5. **Adjust Time Ranges** - Use shorter ranges for troubleshooting, longer for trend analysis

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test dashboard imports
5. Submit a pull request

## License

MIT License - See LICENSE file for details
