---
layout: default
title: ECR
nav_order: 4
---

### AWS ECR Templates


```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: Creates an ECR Repository and a Codebuild job that builds an app whose source is Github.com
Parameters:
  GithubUrl:
    Description: Github URL on Github.com
    Type: String
    Default: 'update'
Resources:
  MyRepository: 
    Type: AWS::ECR::Repository
    Properties: 
      RepositoryName: !Ref 'AWS::StackName'
      ImageScanningConfiguration: 
        ScanOnPush: false
  CodeBuildPolicy:
    Description: Setting IAM policy for service role for CodeBuild
    Properties:
      PolicyDocument:
        Statement:
        - Action:
          - logs:CreateLogGroup
          - logs:CreateLogStream
          - logs:PutLogEvents
          - ecr:*          
          Effect: Allow
          Resource: '*'
      PolicyName: !Join
        - '-'
        -  - !Ref 'AWS::StackName'
           - CodeBuildPolicy
      Roles:
      - !Ref 'CodeBuildRole'
    Type: AWS::IAM::Policy        
  CodeBuildRole:
    Description: Creating service role in IAM for AWS CodeBuild
    Properties:
      AssumeRolePolicyDocument:
        Statement:
        - Action: sts:AssumeRole
          Effect: Allow
          Principal:
            Service: codebuild.amazonaws.com
      Path: /
      RoleName: !Join
        - '-'
        - - !Ref 'AWS::StackName'
          - CodeBuild
    Type: AWS::IAM::Role        
  CodeBuildProject:
    DependsOn:
    - CodeBuildPolicy
    Properties:
      Artifacts:
        Type: no_artifacts    
      Description: !Join
        - ''
        - - 'CodeBuild Project for '
          - !Ref 'AWS::StackName'
      Environment:
        ComputeType: BUILD_GENERAL1_SMALL
        Image: aws/codebuild/amazonlinux2-x86_64-standard:4.0
        Type: LINUX_CONTAINER
        PrivilegedMode: true
        EnvironmentVariables:
        - Name: REPOSITORY
          Type: PLAINTEXT
          Value: !GetAtt MyRepository.RepositoryUri
      Name: !Ref 'AWS::StackName'
      ServiceRole: !Ref 'CodeBuildRole'
      Source:
        Type: GITHUB
        Location: !Ref GithubUrl
        BuildSpec: |
          version: 0.2
          phases:
            install:
              runtime-versions:
                docker: 20
            pre_build:
              commands:
                - echo Logging in to Amazon ECR...
                - aws --version
                - docker login --username <docker-username> --password <docker-token>
                - $(aws ecr get-login --region us-east-1 --no-include-email)
                - REPOSITORY_URI=$REPOSITORY
                - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
                - IMAGE_TAG=${COMMIT_HASH:=latest}
            build:
              commands:
                - echo Build started on `date`
                - echo Building the Docker image...          
                - docker build -t $REPOSITORY_URI:latest .
                - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
            post_build:
              commands:
                - echo Build completed on `date`
                - echo Pushing the Docker images...
                - docker push $REPOSITORY_URI:latest
      Triggers:
        BuildType: BUILD
        Webhook: True
        FilterGroups:
        - - Type: EVENT
            Pattern: PUSH
          - Type: HEAD_REF
            Pattern: ^refs/heads/main$                      
    Type: AWS::CodeBuild::Project
      

```

