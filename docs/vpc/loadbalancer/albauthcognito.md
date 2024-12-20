---
layout: default
title: ALB Authetnication Cognito
parent: Load Balancer
nav_order: 2
---


### AWS ALB Authentication - Cognito

The following template setups up authentication via AWS cognito on an AWS Load Balancer.


```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'ALB Authentication | Cognito'
Parameters:
  VpcId:
    Type: 'AWS::EC2::VPC::Id'
  SubnetIds:
    Type: 'List<AWS::EC2::Subnet::Id>'
  PublicDomainName:
    Type: 'String' # the custom domain name (e.g., grafana.example.com)
  CertificateArn:
    Type: 'String' # the ARN of a ACM certificate valid for the custom domain name
Resources:
  LoadBalancerSecurityGroup: # Security Group for Load Balancer allows incoming HTTPS requests
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: 'Load Balancer'
      VpcId: !Ref VpcId
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        CidrIp: '0.0.0.0/0'
  LoadBalancer:
    Type: 'AWS::ElasticLoadBalancingV2::LoadBalancer'
    Properties:
      SecurityGroups:
      - !Ref LoadBalancerSecurityGroup
      Subnets: !Ref SubnetIds
      Type: application
  Listener:
    Type: 'AWS::ElasticLoadBalancingV2::Listener'
    Properties:
      Certificates:
      - CertificateArn: !Ref CertificateArn
      DefaultActions: # 1. Authenticate against Cognito User Pool
      - Type: 'authenticate-cognito'
        AuthenticateCognitoConfig:
          OnUnauthenticatedRequest: 'authenticate' # Redirect unauthenticated clients to Cognito login page
          Scope: 'openid'
          UserPoolArn: !GetAtt 'UserPool.Arn'
          UserPoolClientId: !Ref UserPoolClient
          UserPoolDomain: !Ref UserPoolDomain
        Order: 1
      - Type: forward # 2. Forward request to target group (e.g., EC2 instances)
        TargetGroupArn: !Ref TargetGroup
        Order: 2
      LoadBalancerArn: !Ref LoadBalancer
      Port: 443
      Protocol: 'HTTPS'
  TargetGroup:
    Type: 'AWS::ElasticLoadBalancingV2::TargetGroup'
    Properties:
      Port: 80
      Protocol: HTTP
      TargetType: ip
      VpcId: !Ref VpcId
  UserPool:
    Type: 'AWS::Cognito::UserPool'
    Properties:
      AdminCreateUserConfig:
        AllowAdminCreateUserOnly: true # Disable self-registration
        InviteMessageTemplate:
          EmailSubject: !Sub '${AWS::StackName}:temporary password'
          EmailMessage: 'Use the username {username} and the temporary password {####} to log in for the first time.'
          SMSMessage: 'Use the username {username} and the temporary password {####} to log in for the first time.'
      AutoVerifiedAttributes:
      - email
      UsernameAttributes:
      - email
      Policies:
        PasswordPolicy:
          MinimumLength: 16
          RequireLowercase: false
          RequireNumbers: false
          RequireSymbols: false
          RequireUppercase: false
          TemporaryPasswordValidityDays: 21
  UserPoolDomain: # Provides Cognito Login Page
    Type: 'AWS::Cognito::UserPoolDomain'
    Properties:
      UserPoolId: !Ref UserPool
      Domain: !Select [2, !Split ['/', !Ref 'AWS::StackId']] # Generates a unique domain name
  UserPoolClient:
    Type: 'AWS::Cognito::UserPoolClient'
    Properties:
      AllowedOAuthFlows:
      - code # Required for ALB authentication
      AllowedOAuthFlowsUserPoolClient: true # Required for ALB authentication
      AllowedOAuthScopes:
      - email
      - openid
      - profile
      - aws.cognito.signin.user.admin
      CallbackURLs:
      - !Sub https://${PublicDomainName}/oauth2/idpresponse # Redirects to the ALB
      GenerateSecret: true
      SupportedIdentityProviders: # Optional: add providers for identity federation
      - COGNITO
      UserPoolId: !Ref UserPool

```