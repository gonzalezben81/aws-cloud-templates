---
layout: default
title: Fargate
parent: AWS Cloud Containerization
nav_order: 2
layout: default
---


### Fargate ECS (Elastic Container Service)


```yml
AWSTemplateFormatVersion: 2010-09-09
Description: An example CloudFormation template for Fargate.
Parameters:
  VPC:
    Type: AWS::EC2::VPC::Id
  SubnetA:
    Type: AWS::EC2::Subnet::Id
  SubnetB:
    Type: AWS::EC2::Subnet::Id
  Certificate:
    Type: String
    # Update with the certificate ARN from Certificate Manager, which must exist in the same region.
    Default: 'arn:aws:acm:region:123456789012:certificate/00000000-0000-0000-0000-000000000000'
  Image:
    Type: String
    # Update with the Docker image. "You can use images in the Docker Hub registry or specify other repositories (repository-url/image:tag)."
    Default: 123456789012.dkr.ecr.region.amazonaws.com/image:tag
  ServiceName:
    Type: String
    # update with the name of the service
    Default: MyService
  ContainerPort:
    Type: Number
    Default: 80
  LoadBalancerPort:
    Type: Number
    Default: 443
  HealthCheckPath:
    Type: String
    Default: /healthcheck
  HostedZoneName:
    Type: String
    Default: company.com
  Subdomain:
    Type: String
    Default: myservice
  # for autoscaling
  MinContainers:
    Type: Number
    Default: 2
  # for autoscaling
  MaxContainers:
    Type: Number
    Default: 10
  # target CPU utilization (%)
  AutoScalingTargetValue:
    Type: Number
    Default: 50
Resources:
  Cluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: !Join ['', [!Ref ServiceName, Cluster]]
  TaskDefinition:
    Type: AWS::ECS::TaskDefinition
    # Makes sure the log group is created before it is used.
    DependsOn: LogGroup
    Properties:
      # Name of the task definition. Subsequent versions of the task definition are grouped together under this name.
      Family: !Join ['', [!Ref ServiceName, TaskDefinition]]
      # awsvpc is required for Fargate
      NetworkMode: awsvpc
      RequiresCompatibilities:
        - FARGATE
      # 256 (.25 vCPU) - Available memory values: 0.5GB, 1GB, 2GB
      # 512 (.5 vCPU) - Available memory values: 1GB, 2GB, 3GB, 4GB
      # 1024 (1 vCPU) - Available memory values: 2GB, 3GB, 4GB, 5GB, 6GB, 7GB, 8GB
      # 2048 (2 vCPU) - Available memory values: Between 4GB and 16GB in 1GB increments
      # 4096 (4 vCPU) - Available memory values: Between 8GB and 30GB in 1GB increments
      Cpu: 256
      # 0.5GB, 1GB, 2GB - Available cpu values: 256 (.25 vCPU)
      # 1GB, 2GB, 3GB, 4GB - Available cpu values: 512 (.5 vCPU)
      # 2GB, 3GB, 4GB, 5GB, 6GB, 7GB, 8GB - Available cpu values: 1024 (1 vCPU)
      # Between 4GB and 16GB in 1GB increments - Available cpu values: 2048 (2 vCPU)
      # Between 8GB and 30GB in 1GB increments - Available cpu values: 4096 (4 vCPU)
      Memory: 0.5GB
      # A role needed by ECS.
      # "The ARN of the task execution role that containers in this task can assume. All containers in this task are granted the permissions that are specified in this role."
      # "There is an optional task execution IAM role that you can specify with Fargate to allow your Fargate tasks to make API calls to Amazon ECR."
      ExecutionRoleArn: !Ref ExecutionRole
      # "The Amazon Resource Name (ARN) of an AWS Identity and Access Management (IAM) role that grants containers in the task permission to call AWS APIs on your behalf."
      TaskRoleArn: !Ref TaskRole
      ContainerDefinitions:
        - Name: !Ref ServiceName
          Image: !Ref Image
          PortMappings:
            - ContainerPort: !Ref ContainerPort
          # Send logs to CloudWatch Logs
          LogConfiguration:
            LogDriver: awslogs
            Options:
              awslogs-region: !Ref AWS::Region
              awslogs-group: !Ref LogGroup
              awslogs-stream-prefix: ecs
  # A role needed by ECS
  ExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Join ['', [!Ref ServiceName, ExecutionRole]]
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service: ecs-tasks.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy'
  # A role for the containers
  TaskRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Join ['', [!Ref ServiceName, TaskRole]]
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service: ecs-tasks.amazonaws.com
            Action: 'sts:AssumeRole'
      # ManagedPolicyArns:
      #   -
      # Policies:
      #   -
  # A role needed for auto scaling
  AutoScalingRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Join ['', [!Ref ServiceName, AutoScalingRole]]
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service: ecs-tasks.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceAutoscaleRole'
  ContainerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Join ['', [!Ref ServiceName, ContainerSecurityGroup]]
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: !Ref ContainerPort
          ToPort: !Ref ContainerPort
          SourceSecurityGroupId: !Ref LoadBalancerSecurityGroup
  LoadBalancerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Join ['', [!Ref ServiceName, LoadBalancerSecurityGroup]]
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: !Ref LoadBalancerPort
          ToPort: !Ref LoadBalancerPort
          CidrIp: 0.0.0.0/0
  Service:
    Type: AWS::ECS::Service
    # This dependency is needed so that the load balancer is setup correctly in time
    DependsOn:
      - ListenerHTTPS
    Properties: 
      ServiceName: !Ref ServiceName
      Cluster: !Ref Cluster
      TaskDefinition: !Ref TaskDefinition
      DeploymentConfiguration:
        MinimumHealthyPercent: 100
        MaximumPercent: 200
      DesiredCount: 2
      # This may need to be adjusted if the container takes a while to start up
      HealthCheckGracePeriodSeconds: 30
      LaunchType: FARGATE
      NetworkConfiguration: 
        AwsvpcConfiguration:
          # change to DISABLED if you're using private subnets that have access to a NAT gateway
          AssignPublicIp: ENABLED
          Subnets:
            - !Ref SubnetA
            - !Ref SubnetB
          SecurityGroups:
            - !Ref ContainerSecurityGroup
      LoadBalancers:
        - ContainerName: !Ref ServiceName
          ContainerPort: !Ref ContainerPort
          TargetGroupArn: !Ref TargetGroup
  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: 10
      # will look for a 200 status code by default unless specified otherwise
      HealthCheckPath: !Ref HealthCheckPath
      HealthCheckTimeoutSeconds: 5
      UnhealthyThresholdCount: 2
      HealthyThresholdCount: 2
      Name: !Join ['', [!Ref ServiceName, TargetGroup]]
      Port: !Ref ContainerPort
      Protocol: HTTP
      TargetGroupAttributes:
        - Key: deregistration_delay.timeout_seconds
          Value: 60 # default is 300
      TargetType: ip
      VpcId: !Ref VPC
  ListenerHTTPS:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - TargetGroupArn: !Ref TargetGroup
          Type: forward
      LoadBalancerArn: !Ref LoadBalancer
      Port: !Ref LoadBalancerPort
      Protocol: HTTPS
      Certificates:
        - CertificateArn: !Ref Certificate
  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      LoadBalancerAttributes:
        # this is the default, but is specified here in case it needs to be changed
        - Key: idle_timeout.timeout_seconds
          Value: 60
      Name: !Join ['', [!Ref ServiceName, LoadBalancer]]
      # "internal" is also an option
      Scheme: internet-facing
      SecurityGroups:
        - !Ref LoadBalancerSecurityGroup
      Subnets:
        - !Ref SubnetA
        - !Ref SubnetB
  DNSRecord:
    Type: AWS::Route53::RecordSet
    Properties:
      HostedZoneName: !Join ['', [!Ref HostedZoneName, .]]
      Name: !Join ['', [!Ref Subdomain, ., !Ref HostedZoneName, .]]
      Type: A
      AliasTarget:
        DNSName: !GetAtt LoadBalancer.DNSName
        HostedZoneId: !GetAtt LoadBalancer.CanonicalHostedZoneID
  LogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: !Join ['', [/ecs/, !Ref ServiceName, TaskDefinition]]
  AutoScalingTarget:
    Type: AWS::ApplicationAutoScaling::ScalableTarget
    Properties:
      MinCapacity: !Ref MinContainers
      MaxCapacity: !Ref MaxContainers
      ResourceId: !Join ['/', [service, !Ref Cluster, !GetAtt Service.Name]]
      ScalableDimension: ecs:service:DesiredCount
      ServiceNamespace: ecs
      # "The Amazon Resource Name (ARN) of an AWS Identity and Access Management (IAM) role that allows Application Auto Scaling to modify your scalable target."
      RoleARN: !GetAtt AutoScalingRole.Arn
  AutoScalingPolicy:
    Type: AWS::ApplicationAutoScaling::ScalingPolicy
    Properties:
      PolicyName: !Join ['', [!Ref ServiceName, AutoScalingPolicy]]
      PolicyType: TargetTrackingScaling
      ScalingTargetId: !Ref AutoScalingTarget
      TargetTrackingScalingPolicyConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ECSServiceAverageCPUUtilization
        ScaleInCooldown: 10
        ScaleOutCooldown: 10
        # Keep things at or lower than 50% CPU utilization, for example
        TargetValue: !Ref AutoScalingTargetValue
Outputs:
  Endpoint:
    Description: Endpoint
    Value: !Join ['', ['https://', !Ref DNSRecord]]

```