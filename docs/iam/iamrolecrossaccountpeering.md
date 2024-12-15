---
layout: default
title: IAM Role Cross Account VPC Peering
parent: Identity Access Management
nav_order: 4
---

### IAM Role Cross Account VPC Peering


The following is an IAM role that allows any principal in a AWS account to accept a VPC peering connection. The condition in the policy only allows the principals that are part of the same AWS Organization or Organizaiton Unit (OU) to accept the peering connection.


```yaml
### References: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/peer-with-vpc-in-another-account.html
AWSTemplateFormatVersion: 2010-09-09
Description: VPC Cross Account Role.
Parameters:
  PrincipalOrgPath:
  Type: String
  Description: AWS Principal Org Path
  Default: "o-/r-/ou-/*"
  RoleName:
  Type: String
  Description: IAM Role Name
  Default: "update"
Resources:  
  peerRole:
    Type: "AWS::IAM::Role"
    Properties:
      RoleName: !Ref Rolename
      AssumeRolePolicyDocument:
        Statement:
          - Principal:
              AWS: "*"
            Action:
              - "sts:AssumeRole"
            Effect: Allow
            Condition:
              ForAnyValue:StringLike:
                aws:PrincipalOrgPaths:
                  - !Ref PrincipalOrgPath
      Path: /
      Policies:
        - PolicyName: root
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action: 
                  - 'ec2:AcceptVpcPeeringConnection'
                  - 'ec2:CreateVpcPeeringConnection'
                Resource: '*'
                
Output:
  peerRoleArn:
    Description: VPC Peering Role Arn
    Value: !GetAtt peerRole.Arn      
```

