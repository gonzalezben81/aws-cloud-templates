---
layout: default
title: IAM
parent: Identity Access Management
nav_order: 1
---

### Identity Access Management


The following is a basic IAM Role with no policies attached. You will need to attach 
the policies in the AWS IAM Console after creating the role. 



```yaml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  BasicIAMRoleName:
    Description: IAM Role Name
    Type: String
    Default: 'update'
Resources:
  IAMRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Ref BasicIAMRoleName
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - ec2.amazonaws.com
          Action:
          - sts:AssumeRole
      Path: "/"
Outputs:
  IAMRoleName:
    Description: IAM Role Name
    Value: !Ref BasicIAMRole
  IAMRoleArn:
    Description: IAM Role Arn
    Value: !GetAtt BasicIAMRole.Arn        
```

