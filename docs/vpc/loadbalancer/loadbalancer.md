---
layout: default
title: LoadBalancer
parent: Load Balancer
nav_order: 1
layout: default
---

### AWS Loadbalancer Templates


#### Basic Load Balancer

```yml
AWSTemplateFormatVersion: 2010-09-09
Description: Basic Load Balancer with ACM Certificate for HTTPS, Security Group, and default Listener.
Parameters:
  VpcId:
    Description: CIDR range for our VPC
    Type: AWS::EC2::VPC::Id
  CidrIp:
    Description: CIDR range for our VPC
    Type: String
    Default: 0.0.0.0/0
  SubnetIds:
    Description: Subnets to connect to ALB
    Type: List<AWS::EC2::Subnet::Id>
Resources:
  LBSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      VpcId: !Ref VpcId
      GroupDescription: SG to allow HTTPS to LoadBalancer
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: -1
          ToPort: -1
          CidrIp: 0.0.0.0/0      
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Ref AWS::StackName
      Scheme: internet-facing
      SecurityGroups:
        - !Ref LBSecurityGroup
      # [Application Load Balancers] You must specify subnets from at least two Availability Zones.
      Subnets:
        - !Select [0, !Split [",", !Ref SubnetIds]]
        - !Select [1, !Split [",", !Ref SubnetIds]]
  MyCertificate:
    Type: "AWS::CertificateManager::Certificate"
    Properties:
    ###This format allows for several sub-domains to wildcard with example.com top-level-domain tld you want to use
      DomainName: '*.something.com'
      ValidationMethod: DNS       
  Listener:
    Type: 'AWS::ElasticLoadBalancingV2::Listener'
    Properties:
      Certificates:
      - CertificateArn: !Ref MyCertificate
      DefaultActions:
        - Type: 'fixed-response' # 1. Authenticate with OICD      
          FixedResponseConfig:
              ContentType: text/html
              MessageBody: 'Incorrect path. Please try Again. Please contact the administrator of the site.'
              StatusCode: 401
      LoadBalancerArn: !Ref ApplicationLoadBalancer
      Port: 443
      Protocol: 'HTTPS'      
Outputs:      
  VPCId:
    Description: vpc id
    Value: !Ref VpcId
  Subnets:
    Description: SubnetId of public subnet A
    Value: !Ref Subnets
  LoadBalancerArn:
    Description: Load Balancer Arn
    Value: !Ref ApplicationLoadBalancer
  LoadBalancerSecurityGroup:
    Description: Security Group of the Load Balancer
    Value: !Ref LBSecurityGroup 
```