---
layout: default
title: TGW Cross Account
parent: Transit Gateway 
nav_order: 2
---

### Transit Gateway (TGW) Cross Account


```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation template to connect private subnets to TGW and update route tables for CIDR in another AWS account

Parameters:
  TransitGatewayId:
    Type: String
    Default: "update"
  # VpcId:
  #   Type: AWS::EC2::VPC::Id
  # SubnetIds:
  #   Type: 'List<AWS::EC2::Subnet::Id>'
  #   Description: "Subnet IDs"    
  PeerAccountCidrBlock:
    Type: String
    Description: The CIDR block of the peer VPC in another AWS account
  RouteTableId1:
    Type: String
    Description: List of Route Table IDs for the private subnets that need updating
  RouteTableId2:
    Type: String
    Description: List of Route Table IDs for the private subnets that need updating    

Resources:
  # Route for each private subnet to route traffic to the TGW for the Peer CIDR block
  UpdatePrivateSubnetsRoute:
    Type: AWS::EC2::Route
    # DependsOn: VPCTransitGatewayAttachment
    Properties:
      DestinationCidrBlock: !Ref PeerAccountCidrBlock
      TransitGatewayId: !Ref TransitGatewayId
      RouteTableId: !Ref RouteTableId1

  # If there are more route tables, you can add additional route updates like this
  UpdatePrivateSubnetsRoute2:
    Type: AWS::EC2::Route
    # DependsOn: VPCTransitGatewayAttachment
    Properties:
      DestinationCidrBlock: !Ref PeerAccountCidrBlock
      TransitGatewayId: !Ref TransitGatewayId
      RouteTableId: !Ref RouteTableId2


```