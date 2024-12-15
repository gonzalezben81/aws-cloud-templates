---
layout: default
title: AWS SSO List s3 buckets
parent: AWS SSO
nav_order: 2
---


### AWS SSO s3 List Buckets


```bash
#!/bin/bash

# List of AWS account profile names
declare -a profiles=("sso_profile_name" "admin")

# CloudFormation stack name
#stack_name="my-cloudformation-stack"

# CloudFormation template file path
#template_file="path/to/your/cloudformation/template.yaml"

# Loop through each profile and launch CloudFormation stack
for profile in "${profiles[@]}"
do
    echo "Listing s3 bucket in $profile"
    
    # Use AWS CLI to launch CloudFormation stack
    aws s3 ls --profile $profile
    aws iam list-account-aliases --profile $profile
       #cloudformation create-stack \
       # --stack-name "$stack_name" \
       # --template-body "file://$template_file" \
       # --profile "$profile"
    
    echo "Listing s3 buckets"
done


```