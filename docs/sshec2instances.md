---
layout: default
title: SSH Between Two ec2 Instances
nav_order: 9
---


### How to ssh between 2 ec2 instances

+ Server One is the source

+ Server Two is the destination

The ec2 instances can be 2 different flavors of linux, e.g. Ubuntu or amazon linux.

Note: You will want to perform these steps on both servers as two way trust will need to be established for the SSH to work.

#### Generate an SSH key

1. Generate an rsa key by running:

```bash
ssh-keygen -t rsa
```

Optionally to save the key directly to another directory run the following code snippet.

```bash
ssh-keygen -t rsa -f /home/test_rsa/id_rsa
```

#### List the ssh key

2. Go to your `~./ssh` folder and you should see two id_rsa keys.

One is a private key and another is a public key.

The public key is designated via the `.pub` file extension.

```bash
total 16
drwxr-xr-x  2 user-name user-name 4096 Oct 24 17:26 .
drwxr-x--- 64 user-name user-name 4096 Oct 24 17:23 ..
-rw-------  1 user-name user-name 2622 Oct 24 17:26 id_rsa
-rw-r--r--  1 user-name user-name  584 Oct 24 17:26 id_rsa.pub
```

#### Copy the ssh key from one server to the other

3. You will want to copy the public key from `Server One` to `Server Two`

You can use nano, vi, or cat to display the id_rsa.pub key that you will place onto the other ec2 instance

+ vi option

```bash
vi id_rsa.pub
```

+ nano option

```bash
nano id_rsa.pub
```

+ cat option

```bash
cat id_rsa.pub
```

Next you will need to manually copy the id_rsa.pub key and then login to the other ec2 instance and place it in the `authorized_keys` file.

<!---

```bash
ssh-copy-id -i ~/.ssh/mykey user@host
```

--->


The code below will append the `id_rsa.pub` file to the end of the `authorized_keys` file on Server One and vice-versa.

```bash
echo <id_rsa.pub> >> authorized_keys
```

Verify the `id_rsa.pub` key is in the `authorized_keys` file.

```bash
cat authorized_keys
```

#### SSH from server one to server two

5. SSH from server one over to server two and vice versa.

Now you should be able to ssh from one instance to another.

```bash
ssh ubuntu@<ip-address>
```

Note: Update ubuntu with the username of the ec2 instance you created. This could be ec2-user if you are using an Amazon Linux image.

#### Example Cloud Formation Templates

Example Security Group (SG) in AWS:

The following security group is an example of a security group that will allow both ec2 instances to connect to each other via SSH.

Note: For troubleshooting purposes you can test the connection via SSH by using the following CIDR Range - `0.0.0.0/0`

```yaml
AWSTemplateFormatVersion: 2010-09-09
Resources:
  EC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: EC2 Security Group
      VpcId: <update-vpc-id>
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: <update-cidr-block e.g. vpc cidr block>

```


<center>References</center>

+ https://dev.to/jeden/connecting-via-ssh-from-one-ec2-instance-to-another-2mk1
+ https://www.ssh.com/academy/ssh/authorized-keys-file
+ https://www.ssh.com/academy/ssh/keygen#what-is-ssh-keygen?

