---
title: AWS Cost Analysis Reports - Usage Type
parent: Cost Analysis Reports
nav_order: 1
layout: default
---

### AWS Cost Analysis Scripts

The following shell script allows you to retrieve unblended costs of the AWS accounts in your AWS Org. This allows you to see the costs of each AWS account and gives a breakdown of the usage type e.g. (Nat Gateway Hours, LoadBalancers, DataTransfer Out). This saves the cost data to a csv file and allows you to easily filter the data to find cost and usage quickly.

Note: This script assumes you are logging in via the AWS SSO CLI, update your login configuration as needed.


Cost Analysis Shell Script:

```bash
#!/bin/bash

# Configurable variables
PROFILE="your-sso-profile"
START_DATE="2025-04-01"
END_DATE="2025-05-01"
CSV_FILE="aws_cost_breakdown_by_usage_type.csv"

# Run AWS Cost Explorer query
response=$(aws ce get-cost-and-usage \
  --profile "$PROFILE" \
  --time-period Start=$START_DATE,End=$END_DATE \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=LINKED_ACCOUNT Type=DIMENSION,Key=USAGE_TYPE \
  --output json)

# Write CSV header
echo "AccountId,UsageType,Amount,Currency" > "$CSV_FILE"

# Parse JSON and write to CSV
echo "$response" | jq -r '
  .ResultsByTime[0].Groups[] |
  [.Keys[0], .Keys[1], .Metrics.UnblendedCost.Amount, .Metrics.UnblendedCost.Unit] |
  @csv' >> "$CSV_FILE"

echo "✅ Detailed cost breakdown saved to: $CSV_FILE"

```    
