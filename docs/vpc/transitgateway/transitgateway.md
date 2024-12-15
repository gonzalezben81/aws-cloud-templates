---
layout: default
title: TGW
parent: Transit Gateway 
nav_order: 1
---


### Transit Gateway

A transit gateway is a hub and spoke model that allows you to more easily peer VPCs together. This could be VPCs in a single account or in multiple accounts. While VPC Peering is very reliable and straightforward, as the number of VPCs being peered together increases; the complexity of the peering increases. 

Enter the transit gateway to help streamline and solve complexity problems. 

The example below has the following features:

+ Auto accept shared attachments
+ AWS Organization and Organizational Unit sharing

```yml
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  AutoAcceptSharedAttachments:
    Type: String
    Default: "enable"
    AllowedValues: 
      - "enable"
      - "disable"
  OrganizationId:
    Type: String
    Default: "update"
    Description: "The AWS Organization ID (or Organizational Unit ID) to share the Transit Gateway with."

Resources:
  myTransitGateway:
    Type: "AWS::EC2::TransitGateway"
    Properties:
      AmazonSideAsn: 65000
      Description: "TGW Connection"
      AutoAcceptSharedAttachments: !Ref AutoAcceptSharedAttachments
      DefaultRouteTableAssociation: "enable"
      DnsSupport: "enable"
      VpnEcmpSupport: "enable"
      Tags:
        - Key: CFStackId
          Value: !Ref 'AWS::StackId'
        - Key: Name
          Value: !Join ["-", [!Ref 'AWS::StackName', "TGW"]]

  myResourceShare:
    Type: "AWS::RAM::ResourceShare"
    DependsOn: myTransitGateway
    Properties:
      Name: !Join ["-", [!Ref 'AWS::StackName', "TGW Resource Share"]]
      AllowExternalPrincipals: false
      Principals:
        - !Sub "arn:aws:organizations::${AWS::AccountId}:organization/${OrganizationId}"
      ResourceArns:
        - !Sub arn:aws:ec2:${AWS::Region}:${AWS::AccountId}:transit-gateway/${myTransitGateway}  # Fixed use of !Ref inside !Sub
      Tags:
        - Key: CFStackId
          Value: !Ref 'AWS::StackId'
        - Key: Name
          Value: !Join ["-", [!Ref 'AWS::StackName', "resourceshare"]]

Outputs:
  TransitGatewayId:
    Value: !Ref myTransitGateway
    Description: "The ID of the created Transit Gateway."
  ResourceShareArn:
    Value: !GetAtt myResourceShare.Arn
    Description: "Arn or the Resource Share."
  ResourceShareId:
    Value: !Ref myResourceShare
    Description: "ID of the Resource Share."     


```