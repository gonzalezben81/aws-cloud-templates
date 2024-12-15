---
layout: default
title: VPC
---

### AWS VPC Templates

Basic VPC template:

```yml
AWSTemplateFormatVersion: 2010-09-09
Description: Custom VPC Creation With Private and Public Subnet
Parameters:
  VPCName:
    Description: Name of the VPC
    Type: String
    Default: DemoCustomVPC
  VPCCidr:
    Description: CIDR range for our VPC
    Type: String
    Default: 10.97.222.0/25
  PrivateSubnetACidr:
    Description: Private Subnet IP Range
    Type: String
    Default: 10.97.222.0/27
  PrivateSubnetBCidr:
    Description: Private Subnet IP Range
    Type: String
    Default: 10.97.222.32/27
  PublicSubnetACidr:
    Description: Public Subnet IP Range
    Type: String
    Default: 10.97.222.64/27
  PublicSubnetBCidr:
    Description: Public Subnet IP Range
    Type: String
    Default: 10.97.222.96/27
  AvailabilityZoneA:
    Description: Avaibalbility Zone 1
    Type: String
    Default: us-east-1a
  AvailabilityZoneB:
    Description: Avaibalbility Zone 2
    Type: String
    Default: us-east-1b
Resources:
  DemoVPC:
    Type: 'AWS::EC2::VPC'
    Properties:
      EnableDnsSupport: true
      EnableDnsHostnames: true
      CidrBlock: !Ref VPCCidr
      Tags:
        - Key: Name
          Value: !Ref VPCName
  PrivateSubnetA:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref DemoVPC
      AvailabilityZone: !Ref AvailabilityZoneA
      CidrBlock: !Ref PrivateSubnetACidr
      Tags:
        - Key: Name
          Value: !Sub '${VPCName}-PrivateSubnetA'
  PrivateSubnetB:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref DemoVPC
      AvailabilityZone: !Ref AvailabilityZoneB
      CidrBlock: !Ref PrivateSubnetBCidr
      Tags:
        - Key: Name
          Value: !Sub '${VPCName}-PrivateSubnetB'
  PublicSubnetA:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref DemoVPC
      AvailabilityZone: !Ref AvailabilityZoneA
      CidrBlock: !Ref PublicSubnetACidr
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${VPCName}-PublicSubnetA'
  PublicSubnetB:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref DemoVPC
      AvailabilityZone: !Ref AvailabilityZoneB
      CidrBlock: !Ref PublicSubnetBCidr
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${VPCName}-PublicSubnetB'
  InternetGateway:
    Type: 'AWS::EC2::InternetGateway'
  VPCGatewayAttachment:
    Type: 'AWS::EC2::VPCGatewayAttachment'
    Properties:
      VpcId: !Ref DemoVPC
      InternetGatewayId: !Ref InternetGateway
  PublicRouteTable:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref DemoVPC
  PublicRoute:
    Type: 'AWS::EC2::Route'
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  PublicSubnetRouteTableAssociationA:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PublicSubnetA
      RouteTableId: !Ref PublicRouteTable
  PublicSubnetRouteTableAssociationB:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PublicSubnetB
      RouteTableId: !Ref PublicRouteTable
  ElasticIPA:
    Type: 'AWS::EC2::EIP'
    Properties:
      Domain: vpc
  ElasticIPB:
    Type: 'AWS::EC2::EIP'
    Properties:
      Domain: vpc
  NATGatewayA:
    Type: 'AWS::EC2::NatGateway'
    Properties:
      AllocationId: !GetAtt 
        - ElasticIPA
        - AllocationId
      SubnetId: !Ref PublicSubnetA
  NATGatewayB:
    Type: 'AWS::EC2::NatGateway'
    Properties:
      AllocationId: !GetAtt 
        - ElasticIPB
        - AllocationId
      SubnetId: !Ref PublicSubnetB
  PrivateRouteTableA:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref DemoVPC
      Tags:
        - Key: Name
          Value: !Sub '${VPCName}-PrivateRouteTableA'      
        - Key: subnet-association
          Value: !Sub '${VPCName}-PrivateSubnetA'
  PrivateRouteTableB:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref DemoVPC
      Tags:
        - Key: Name
          Value: !Sub '${VPCName}-PrivateRouteTableB'      
        - Key: subnet-association
          Value: !Sub '${VPCName}-PrivateSubnetB'    
  PrivateRouteToInternetA:
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PrivateRouteTableA
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NATGatewayA
  PrivateRouteToInternetB:
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PrivateRouteTableB
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NATGatewayB
  PrivateSubnetRouteTableAssociationA:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PrivateSubnetA
      RouteTableId: !Ref PrivateRouteTableA
  PrivateSubnetRouteTableAssociationB:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PrivateSubnetB
      RouteTableId: !Ref PrivateRouteTableB
Outputs:
  VPCId:
    Description: vpc id
    Value: !Ref DemoVPC
  PublicSubnetA:
    Description: SubnetId of public subnet A
    Value: !Ref PublicSubnetA
  PublicSubnetB:
    Description: SubnetId of public subnet B
    Value: !Ref PublicSubnetB
  PrivateSubnetA:
    Description: SubnetId of private subnet A
    Value: !Ref PrivateSubnetA
  PrivateSubnetB:
    Description: SubnetId of private subnet B
    Value: !Ref PublicSubnetB
```    
