---
layout: default
title: AWS SSO IAM Roles
parent: AWS SSO
nav_order: 3
---


### AWS SSO IAM Roles


```bash
#!/bin/bash

# List of AWS account profile names
declare -a profiles=("sso_profile_name" "admin")

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