# Lab 2: Create VPC, EC2, and RDS via CloudFormation

## Overview

This lab activity will demonstrate how to create an EC2 instance using AWS CloudFormation. As a challenge, you will also create an RDS instance using CloudFormation.

---

## Prerequisites

- You must have created your own AWS EC2 key pair beforehand.
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured.
- A text editor (e.g., VS Code, nano, vim) and access to your terminal (local, EC2, or [AWS CloudShell](https://docs.aws.amazon.com/cloudshell/latest/userguide/)).
- Permissions to create CloudFormation stacks, S3 buckets, EC2, and RDS instances.

---

## 1. Create a VPC with Public and Private Subnets in CloudFormation

### 1-a. Prepare the VPC CloudFormation Template

1. In your terminal, create a new file called `vpc-subnets.yml`.
2. Copy the contents from the gist below and save to your file:  
   [https://gist.github.com/carl-alarcon/80fe39e66845cc9e3f7e85a155058818](https://gist.github.com/carl-alarcon/80fe39e66845cc9e3f7e85a155058818)

---

### 1-b. Create an S3 Bucket to Store Your Templates

Replace `<YOUR_NAME>` with your own unique name.

```
aws s3 mb s3://cdmp.<YOUR_NAME> --region ap-southeast-1
```

1-c. Upload vpc-subnets.yml to Your S3 Bucket

```
aws s3 cp vpc-subnets.yml s3://cdmp.<YOUR_NAME>
```

1-d. Get the S3 Object URL for the Template

Open the S3 console, browse to your bucket, select your vpc-subnets.yml, and copy the object URL.

1-e. Create the VPC Stack via CloudFormation Console

Go to the AWS CloudFormation Console.

Click Create stack → With new resources (standard).

For "Template source", select Amazon S3 URL and paste the copied S3 object URL.


1-f. Enter Stack Details

```
Stack name: VPC-<YOUR_NAME>
EnvironmentName: Development-<YOUR_NAME>
```

1-g. Create the Stack

Leave all remaining configuration as default and create your VPC stack.

2. Create an EC2 Instance via CloudFormation

2-a. Prepare the EC2 CloudFormation Template

Create a new file named ec2.yml.

Copy the contents from the gist below:
```
Description:
  Creates an Amazon EC2 instance in a public subnet.

Parameters:
  CloudFormationVPC:
    Description: The CloudFormation Stack that contains the VPC and subnets.
    Type: String
  KeyPair:
    Description: Key pair to use.
    Type: "AWS::EC2::KeyPair::KeyName"
  AmazonMachineImageId:
    Description: AMI Image ID
    Type: String
    Default: ami-0c6ebb5b9bce4ba15

Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref AmazonMachineImageId
      InstanceType: t2.micro
      KeyName: !Ref KeyPair
      NetworkInterfaces:
        - GroupSet:
          - !ImportValue 
            'Fn::Sub': '${CloudFormationVPC}-SecurityGroupID'
          AssociatePublicIpAddress: 'true'
          DeviceIndex: '0'
          SubnetId: !ImportValue 
            'Fn::Sub': '${CloudFormationVPC}-PublicSubnetID'
```


2-b. Upload ec2.yml to S3
```
aws s3 cp ec2.yml s3://cdmp.<YOUR_NAME>
```

2-c. Launch the EC2 Stack in CloudFormation

Get the S3 object URL for ec2.yml as in step 1-d.

In the CloudFormation console, start "Create stack".

For Step 1, enter the template S3 URL for your ec2.yml.

For Step 2, enter these parameters:
```
Stack Name: EC2-<YOUR_NAME>
CloudFormationVPC: Name of your VPC stack (e.g., VPC-<YOUR_NAME>)
KeyPair: Name of your EC2 key pair
```

Leave the remaining configuration as default.

2-d. Create the EC2 Stack

Finish stack creation and verify your EC2 instance launches in your specified VPC.

3. ☑️ Challenge: Create your Amazon RDS Instance
You are challenged to create an Amazon RDS t2.micro instance using CloudFormation on your own.

Reference: AWS::RDS::DBInstance Doc

Step 1: Create Your RDS CloudFormation Template
Create a new file called rds.yml in your workspace or local directory, and paste the following sample template for an RDS MySQL database (using t2.micro instance class):

```
AWSTemplateFormatVersion: '2010-09-09'
Description: Create a MySQL RDS t3.micro instance

Parameters:
  DBUsername:
    Description: The database admin account username
    Type: String
    AllowedPattern: "[a-zA-Z][a-zA-Z0-9]*"
    MinLength: 1
    MaxLength: 16
    ConstraintDescription: must begin with a letter and contain only alphanumeric characters.
  DBPassword:
    Description: The database admin account password
    NoEcho: true
    Type: String
    MinLength: 8
    MaxLength: 41
    ConstraintDescription: must be between 8 and 41 characters.

Resources:
  MyDBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Subnet group for RDS instance
      SubnetIds:
        - subnet-<># Make sure this is a private subnet in us-east-1a (for example)
        - subnet-<> # And this is a private subnet in us-east-1b (for example)

  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow MySQL access
      VpcId: <> # Replace with your VPC ID
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          CidrIp: 0.0.0.0/0   # (For test/demo only; restrict in production!)

  MyRDSInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: myrds-<NAME>
      Engine: mysql
      # Optionally, specify a recent engine version. Otherwise, the latest available will be used.
      # EngineVersion: 8.0.36
      DBInstanceClass: db.t3.micro
      AllocatedStorage: 20
      MasterUsername: !Ref DBUsername
      MasterUserPassword: !Ref DBPassword
      VPCSecurityGroups:
        - !GetAtt MySecurityGroup.GroupId
      DBSubnetGroupName: !Ref MyDBSubnetGroup
      PubliclyAccessible: true
      BackupRetentionPeriod: 0
      MultiAZ: false

Outputs:
  RDSInstanceEndpoint:
    Description: RDS Endpoint
    Value: !GetAtt MyRDSInstance.Endpoint.Address
```

Important:
You must replace these lines with your actual resources:

subnet-xxxxxx1, subnet-xxxxxx2: IDs of private subnets from your VPC
vpc-xxxxxx: ID of your VPC
You can find these values in the CloudFormation Outputs of your VPC stack, or in the AWS Console.

Step 2: Upload the Template to S3
```
aws s3 cp rds.yml s3://cdmp.<YOUR_NAME>
```

Step 3: Get S3 Object URL
In the S3 Console, find your rds.yml file in your bucket, and copy the object URL (as you did with the other templates).

Step 4: Launch the RDS Stack in CloudFormation Console

Go to the CloudFormation Console.

Create stack → With new resources (standard).

Use Amazon S3 URL and paste your RDS template S3 URL.

For parameters, enter your DB username and password.

Leave the rest as defaults.

Click Next through the final steps and create the stack.

Step 5: Update Your Submission JSON

Copy and paste your information in this format:
```
{
  "ec2_cloudformation_template_url": "https://s3.amazonaws.com/cdmp.<YOUR_NAME>/ec2.yml",
  "cloudformation_s3_bucket_arn": "arn:aws:s3:::cdmp.<YOUR_NAME>",
  "rds_cloudformation_template_url": "https://s3.amazonaws.com/cdmp.<YOUR_NAME>/rds.yml"
}
```

