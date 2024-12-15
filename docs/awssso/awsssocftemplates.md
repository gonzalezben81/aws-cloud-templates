---
layout: default
title: AWS SSO CF Templates Shell Script
parent: AWS SSO
nav_order: 
---


### AWS SSO CF Templates Shell Script


```bash
#!/bin/bash

# List of AWS account profile names
declare -a profiles=("sso_profile_name" "admin")

# CloudFormation stack name
stack_name="my-cloudformation-stack"

# CloudFormation template file path
template_file="path/to/your/cloudformation/template.yaml"

# Loop through each profile and launch the CloudFormation stack(s)
for profile in "${profiles[@]}"
do
    echo "Launching Cloud Formation in $profile"
    
    # Use AWS CLI to launch CloudFormation stack
    aws cloudformation create-stack \
       --stack-name "$stack_name" \
       --template-body "file://$template_file" \
       --profile "$profile"

    echo "Success!!! Cloud Formatin Launched"
done


```