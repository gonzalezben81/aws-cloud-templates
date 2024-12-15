---
layout: default
title: AWS-SSO
parent: AWS SSO
nav_order: 1
---


### AWS SSO Templates

The following template adds an `admin` level user to an AWS Account in your AWS Organization. The permission set attached to the user has admin level permissions and gives the user near root level access. The output of the cloud formation template gives the assignment status and the arn of the permission set for this particular user. 

*Note: The user must exist in your OIDC provider e.g. Microsoft Entra ID (Azure).*

Parameters needed:

+ AWS Account ID
+ User Email
+ Permission Set Name
+ SSO Instance Arn
+ Identity Store ID

Outputs:
+ Permission Set Arn

```yml
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  AccountId:
    Type: String
    Description: "The AWS account ID to which the permission set will be assigned."
    Default: "update"
  UserEmail:
    Type: String
    Description: "The email of the SSO user to whom the permission set will be assigned."
    Default: "update"
  PermissionSetName:
    Type: String
    Description: Name of Permission Set assigned to User
    Default: "Admin Permissions"
  SSOInstanceArn:
    Type: String
    Description: Arn of SSO Instance
    Default: "update"
  IdentityStoreId:
    Type: String
    Description: Arn of SSO Instance
    Default: "update"    
Resources:
  SSOPermissionSet:
    Type: "AWS::SSO::PermissionSet"
    Properties: 
      InstanceArn: !Sub "arn:aws:sso:::instance/ssoins-${SSOInstanceArn}"  # Replace with your actual SSO Arn
      Name: !Join ["-",[ !Ref AWS::StackName ,"SSOUserPolicy"]]
      Description: "Permission set for Admin access"
      ManagedPolicies: 
        - "arn:aws:iam::aws:policy/AdministratorAccess"
      SessionDuration: "PT8H"
      Tags:
        - Key: "Permission Set User"
          Value: !Ref UserEmail
  SSOLookUpIAMRole:
    Type: AWS::IAM::Role
    Properties:
      Description: "Retrieves SSO instance information"
      RoleName: !Join ["-",[ !Ref AWS::StackName ,"SSOAdminRoleCreator"]]
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - ec2.amazonaws.com
          Action:
          - sts:AssumeRole
        - Effect: Allow
          Principal:
            Service:
            - lambda.amazonaws.com
          Action:
          - sts:AssumeRole          
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AWSSSOReadOnly
      Policies:
        - PolicyName: "LambdaCloudWatchLogPermissions"
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - identitystore:ListUsers
                  - identitystore:DescribeUser
                Resource: "*"             
              - Effect: Allow
                Action:
                  - logs:*
                Resource: arn:aws:logs:*:*:*        
  UserLookupFunction:
    Type: "AWS::Lambda::Function"
    Properties:
      Handler: "index.handler"
      Role: !GetAtt SSOLookUpIAMRole.Arn
      Runtime: python3.12
      Timeout: 900
      Environment:
        Variables:
          identity_store_id: !Ref IdentityStoreId
          sso_instance_arn: !Ref SSOInstanceArn
          user_email: !Ref UserEmail
      Code:
        ZipFile: |
          import json
          import cfnresponse
          import boto3
          import os
          
          client = boto3.client('identitystore')
          
          identity_store_id = os.getenv('identity_store_id')
          sso_instance_arn = os.getenv('sso_instance_arn')
          user_email = os.getenv('user_email')
          try:
              def handler(event, context):
                if event['RequestType'] == "Create":
                  user_list =client.list_users(IdentityStoreId=identity_store_id)
                  for user in user_list['Users']:
                    response = client.describe_user(IdentityStoreId=identity_store_id,UserId=user['UserId'])
                    if 'Emails' in response:
                      if response['Emails'][0]['Value'] == user_email:
                        print(response['UserId'])
                        sso_user_id = response['UserId']
                      cfnresponse.send(event, context, cfnresponse.SUCCESS,{'UserID': str(sso_user_id)})
                elif event['RequestType'] == "Delete":
                  print("Deleting Resources")
                  cfnresponse.send(event, context, cfnresponse.SUCCESS,{'Deleted': 'Deleted'})
          except Exception as e:
            cfnresponse.send(event, context, cfnresponse.FAILED,{'Message',str(e)})
  UserLookup:
    Type: "Custom::UserLookup"
    Version: "1.0"
    Properties:
      ServiceToken: !GetAtt UserLookupFunction.Arn

  SSOPermissionSetAssignment:
    Type: "AWS::SSO::Assignment"
    Properties:
      InstanceArn: !Sub "arn:aws:sso:::instance/ssoins-${SSOInstanceArn}"
      PermissionSetArn: !GetAtt SSOPermissionSet.PermissionSetArn
      PrincipalId: !GetAtt UserLookup.UserID # This gets the resolved user ID or GUID (Globally Unique Identifier)
      PrincipalType: "USER"
      TargetId: !Ref AccountId
      TargetType: "AWS_ACCOUNT"

Outputs:
  PermissionSetArn:
    Value: !GetAtt SSOPermissionSet.PermissionSetArn
    Description: "The ARN of the created AWS SSO permission set."

  AssignmentStatus:
    Value: "Assigned"
    Description: "The permission set has been assigned to the user."

```