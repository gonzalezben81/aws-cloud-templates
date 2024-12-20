---
layout: default
title: ec2 Transit Gateway
parent: ec2 Elastic Compute Cloud
nav_order: 4
---

### ec2 TGW Example

The following is a template for testing the connection between two ec2 instances connected by a transit gateway. 


```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation template to create a Security Group, an EC2 instance, and allow traffic from another accounts VPC CIDR

Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: VPC ID where the security group will be created
  SubnetId:
    Type: AWS::EC2::Subnet::Id
    Description: Subnet ID where the EC2 instance will be launched
  SecurityGroupName:
    Type: String
    Description: Name of the Security Group
  PeerAccountCidrBlock:
    Type: String
    Description: CIDR block of the VPC in the opposite account that will be allowed access
  InstanceType:
    Type: String
    Default: t3.medium
    Description: EC2 instance type

Resources:
  # Create a Security Group
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security Group allowing access from another accounts VPC
      VpcId: !Ref VpcId
      GroupName: !Ref SecurityGroupName
      Tags:
        - Key: Name
          Value: !Ref SecurityGroupName
  # Add Ingress Rule to the Security Group for the peer account's CIDR
  MySecurityGroupIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref MySecurityGroup
      IpProtocol: tcp
      FromPort: 22  # Change as necessary
      ToPort: 22    # Change as necessary
      CidrIp: !Ref PeerAccountCidrBlock
  # Create an IAM Role for EC2 with SSM Access
  MyIAMRole:
    Type: AWS::IAM::Role
    Properties: 
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
        - arn:aws:iam::aws:policy/AmazonSSMFullAccess
        - arn:aws:iam::aws:policy/CloudWatchFullAccess        
      Path: "/"
      Policies:
        - PolicyName: SSMAccessPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - ssm:*
                  - ec2messages:*
                  - cloudwatch:PutMetricData
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Resource: "*"

  # Create an Instance Profile for the IAM Role
  MyInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Roles:
        - !Ref MyIAMRole
      Path: "/"      

  # Create an EC2 instance
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-0a21b6ec028435788 # Replace with a valid AMI ID for your region
      SubnetId: !Ref SubnetId
      IamInstanceProfile: !Ref MyInstanceProfile
      SecurityGroupIds:
        - !Ref MySecurityGroup
      Tags:
        - Key: Name
          Value: MyEC2Instance

Outputs:
  SecurityGroupId:
    Description: The ID of the created Security Group
    Value: !Ref MySecurityGroup
  EC2InstanceId:
    Description: The ID of the created EC2 instance
    Value: !Ref MyEC2Instance


```