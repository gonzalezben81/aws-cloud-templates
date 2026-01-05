---
layout: default
title: s3 Bucket Replication
parent: s3
nav_order: 1
---

# s3 Bucket Replication

AWS s3 has a very unique feature in that you can automatically back up an s3 bucket to another bucket in the same region or a different region. This helps to ensure the objects stored in s3 can be closer to users or backed up in case of an outage. Below are two yaml templates that allow you to create a `source bucket` and a `destination bucket`. 

## s3 Desination Bucket

The following `yml` template and allows a user to create an destination s3 bucket for an s3 source bucke to replicate objects to. 

```yml
AWSTemplateFormatVersion: '2010-09-09'
Description: S3 Destination Bucket

Resources:
  DestinationBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub ${AWS::StackName}-destbucket-${AWS::AccountId}
      PublicAccessBlockConfiguration:
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      VersioningConfiguration:
        Status: Enabled

Outputs:
  DestinationBucketName:
    Value: !Ref DestinationBucket
    Description: Destination S3 Bucket Name
```

## s3 Source Bucket with replication


The following `yml` template and allows a user to create an s3 bucket that replicates to a destination bucket. 

One important note for this template is the bucket policy. The explicit deny requires any s3 action to utilize a secure HTTPS connection and denies any s3 actions if HTTP is utilized. 

In summary the yml template creates the following resources in AWS.

+ IAM Role that allows s3 bucket replication to the destination bucket
+ Creates an s3 `source bucket` that will replicate to the destination bucket
+ Creates an s3 `bucket policy` that requires a secure transport e.g. HTTPS connection

*Note: You will need to create the s3 destination bucket before utilizing this template*

```yml
AWSTemplateFormatVersion: '2010-09-09'
Description: S3 Cross-Region Replication with Encryption and IAM Role
Parameters:
  DestBucket:
    Type: String

Resources:
  s3BucketReplicationRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - s3.amazonaws.com
            Action:
              - sts:AssumeRole
      Policies:
        - PolicyName: S3ReplicationPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - s3:GetObject
                  - s3:GetObjectVersion
                  - s3:GetBucketVersioning
                  - s3:ListBucket
                  - s3:GetObjectVersionAcl
                  - s3:GetObjectVersionTagging
                  - s3:ObjectOwnerOverrideToBucketOwner
                Resource:
                  - !Sub arn:aws:s3:::${AWS::StackName}-sourcebucket-${AWS::AccountId}
                  - !Sub arn:aws:s3:::${AWS::StackName}-sourcebucket-${AWS::AccountId}/*
              - Effect: Allow
                Action:
                  - s3:ReplicateObject
                  - s3:ReplicateDelete
                  - s3:ReplicateTags
                Resource:
                  - !Join
                    - ''
                    - - !Ref DestBucket
                      - /*

  SourceBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub ${AWS::StackName}-sourcebucket-${AWS::AccountId}
      VersioningConfiguration:
        Status: Enabled
      PublicAccessBlockConfiguration:
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      ReplicationConfiguration:
        Role: !GetAtt s3BucketReplicationRole.Arn
        Rules:
          - Id: s3BucketReplicationRule1
            Status: Enabled
            Prefix: ''
            Destination:
              Bucket: !Ref DestBucket #!Sub "arn:aws:s3:::${AWS::StackName}-destbucket-${AWS::AccountId}"
              StorageClass: STANDARD

  SourceBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref SourceBucket
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Sid: RequireEncryptionInTransit
            Effect: Deny
            Principal: '*'
            Action:
              - s3:GetObject
              - s3:PutObject
              - s3:ListBucket
            Resource:
              - !Sub arn:aws:s3:::${AWS::StackName}-sourcebucket-${AWS::AccountId}
              - !Sub arn:aws:s3:::${AWS::StackName}-sourcebucket-${AWS::AccountId}/*
            Condition:
              Bool:
                aws:SecureTransport: 'false'

Outputs:
  SourceBucketName:
    Value: !Ref SourceBucket
    Description: Source S3 Bucket Name

  ReplicationRoleArn:
    Value: !GetAtt s3BucketReplicationRole.Arn
    Description: IAM Role ARN for S3 Replication
```