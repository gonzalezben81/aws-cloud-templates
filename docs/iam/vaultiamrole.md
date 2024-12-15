---
layout: default
title: Hashicorp Vault IAM Role
parent: Identity & Access Management
nav_order: 3
---

### IAM Role for connecting Hashicorp Vault

The following is a template for connecting AWS to your Hashciorp Vault for authenticating to AWS.


```yaml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  VaultIAMRoleName:
    Description: IAM Role Name for interacting with Vault
    Type: String
    Default: vaultauthentication
Resources:
  VaultIAMRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Ref VaultIAMRoleName
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
  VaultIAMRolePolicies:
    Type: AWS::IAM::Policy
    Properties:
      PolicyName: vaultiamroleauth
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action:
              - ec2:DescribeInstances
              - iam:GetInstanceProfile
              - iam:GetUser
              - iam:GetRole
            Resource: '*'
          - Effect: Allow
            Action:
              - sts:AssumeRole
            Resource:
              - !Join ['',['arn:aws:iam::',!Ref AWS::AccountId,':role/',!Ref VaultIAMRoleName]]
          - Sid: ManageOwnAccessKeys
            Effect: Allow
            Action:
              - iam:CreateAccessKey
              - iam:DeleteAccessKey
              - iam:GetAccessKeyLastUsed
              - iam:GetUser
              - iam:ListAccessKeys
              - iam:UpdateAccessKey
            Resource:
              - !Join ['',['arn:aws:iam::*:user/',!Ref VaultIAMRoleName]]
      Roles:
      - !Ref VaultIAMRole
  VaultIAMInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName: !Ref VaultIAMRoleName
      Path: "/"
      Roles:
      - !Ref VaultIAMRole
Outputs:
  IAMRoleName:
    Description: IAM Role Name
    Value: !Ref VaultIAMRole
  IAMRoleArn:
    Description: IAM Role Arn
    Value: !GetAtt VaultIAMRole.Arn        
```