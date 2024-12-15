---
layout: default
title: EC2: Elastic Compute Cloud
parent: ec2 Elastic Compute Cloud
nav_order: 1
---

### AWS ec2 Templates


The following template creates a basic ec2 instance. 



List of packages installed in the ec2 (can be removed):
+ R
+ python3
+ git
+ docker


Basic ec2 Instance:

```yaml
  ec2InstanceTwo:
    Type: 'AWS::EC2::Instance'
    Properties:
      ImageId: ami-0481e8ba7f486bd99 #ami-01d08089481510ba2
      InstanceType: t3.micro
      IamInstanceProfile: !Ref RstudioInstanceProfile
      AvailabilityZone: !Ref AvailabilityZoneA
      SubnetId: !Ref PublicSubnetA
      KeyName: vpc-one
      SecurityGroupIds:
        - !Ref DemoSecurityGroup
        - !Ref LBSecurityGroup
      UserData: !Base64 
        'Fn::Sub': |
          #!/bin/bash
          sudo apt-get update -y
          sudo apt-get install nginx -y
          sudo apt-get install r-base-core -y
          sudo apt-get install r-base-dev -y
          sudo apt-get install git -y
          sudo apt-get install docker.io -y
          sudo groupadd docker
          sudo usermod -aG docker $USER
          sudo newgrp docker
          sudo apt-get install -y python3
          sudo apt-get update -y
          sudo wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb =o /tmp/amazon-cloudwatch-agent.deb
          sudo dpkg -i /tmp/amazon-cloudwatch-agent.deb
          cat << EOF > /opt/aws/amazon-cloudwatch-agent/bin/config.json
          {
	          "agent": {
	            "metrics_collection_interval": 10,
	            "run_as_user": "root"
	          },
	              "logs": {
	                "logs_collected": {
	              	"files": {
	              	  "collect_list": [
	              		{
	              		  "file_path": "/var/log/syslog",
	              		  "log_group_name": "${EC2LogGroup}",
	              		  "log_stream_name": "${EC2LogGroup}/var/log/syslog",
	              		  "timezone": "Local"
	              		  }
	              		  ]
	              	  }
	                }
	              }
              }
          EOF
          sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
      Tags:
        - Key: Name
          Value: !Ref VPCName 

```