---
layout: default
title: Hashicorp Vault IAM User
parent: Identity & Access Management
nav_order: 2
---

### IAM User for connecting Hashicorp Vault

The following is a template for connecting AWS to your Hashciorp Vault for authenticating to AWS.


```yaml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  VaultIAMUserName:
    Description: IAM User Name for interacting with Vault
    Type: String
    Default: vaultauthentication
Resources:
  VaultIAMUser:
    Type: AWS::IAM::User
    Properties:
      UserName: !Ref VaultIAMUserName
      Path: "/"
      Policies:
      - PolicyName: vaultiamuserauth
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - ec2:DescribeInstances
                - iam:GetInstanceProfile
                - iam:GetUser
                - iam:ListRoles
                - iam:GetRole
                - iam:CreateUser
                - iam:PutUserPolicy
              Resource: '*'
Outputs:
  IAMUserName:
    Description: IAM User Name
    Value: !Ref VaultIAMUser
  IAMUserArn:
    Description: IAM User Arn
    Value: !GetAtt VaultIAMUser.Arn   
```    
