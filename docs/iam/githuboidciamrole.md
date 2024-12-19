---
layout: default
title: GitHub OIDC IAM Role
parent: Identity & Access Management
nav_order: 5
---


```yaml
Parameters:
  GitHubOrg:
    Description: Name of GitHub organization/user (case sensitive)
    Type: String
  RepositoryName:
    Description: Name of GitHub repository (case sensitive)
    Type: String
  OIDCProviderArn:
    Description: Arn for the GitHub OIDC Provider.
    Default: "arn:aws:iam::<account-number>:oidc-provider/token.actions.githubusercontent.com"
    Type: String
  OIDCAudience:
    Description: Audience supplied to configure-aws-credentials.
    Default: "sts.amazonaws.com"
    Type: String

Conditions:
  CreateOIDCProvider: !Equals 
    - !Ref OIDCProviderArn
    - ""

Resources:
  Role:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Action: sts:AssumeRoleWithWebIdentity
            Principal:
              Federated: !If 
                - CreateOIDCProvider
                - !Ref GithubOidc
                - !Ref OIDCProviderArn
            Condition:
              StringEquals:
                token.actions.githubusercontent.com:aud: !Ref OIDCAudience
              StringLike:
                token.actions.githubusercontent.com:sub: !Sub repo:${GitHubOrg}/${RepositoryName}:*

  GithubOidc:
    Type: AWS::IAM::OIDCProvider
    Condition: CreateOIDCProvider
    Properties:
      Url: https://token.actions.githubusercontent.com
      ClientIdList: 
        - sts.amazonaws.com
      ThumbprintList:
        - ffffffffffffffffffffffffffffffffffffffff

Outputs:
  Role:
    Value: !GetAtt Role.Arn 
```


### Example policies to attach to AWS GitHub Actions OIDC Role

> *Note: If you see the following: An error occurred (AccessDenied) when calling the ListBuckets operation: User: arn:aws:sts::<account-number>:assumed-role/<role-name>/GitHubActions is not authorized to perform: s3:ListAllMyBuckets because no identity-based policy allows the s3:ListAllMyBuckets action: your policy has failed and you need to update the policy for the role in AWS.*

### Minimal AWS GitHub Actions :octocat:


The following minimal GitHub Actions example shows how you can login to AWS via the GitHub OIDC role you setup with the above template. You can place the AWS IAM Role Arn as a `secret` in the GitHub environment an retrieve it via the ```yaml${{ secrets.<secret-name> }}``` from the GitHub secrets in the repo. Additonally this example simply describes the ec2 instances available in `us-east-1` region in AWS. Simply update the region to your specific region to utilize the AWS CLI commands in that particular region.

Currently this GitHub actions workflow is triggered via a push to the `main` branch and also allows for a manual trigger via `workflow_dispatch`.

```yaml
name: AWS GitHub Actions Test

on: 
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout the GitHub Repository
      uses: actions/checkout@v4    
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-region: us-east-1
        role-to-assume: ${{ secrets.AWS_GITHUB_ACTIONS }}
    - name: Describe ec2 instances
      run: |
       aws ec2 describe-instances
```

#### References

[GitHub OIDC References](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)

[How to use GitHub OIDC with AWS in GitHub Actions](https://github.com/aws-actions/configure-aws-credentials)
