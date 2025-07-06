## Overview: 

This lab activity will demonstrate how to create an EC2 instance using CloudFormation. As a challenge, you will be asked to create an RDS instance using CloudFormation also.

Prerequisite

- Before doing this lab, you should have your own keypair.

## 1. Create a VPC with public and private subnets in AWS CloudFormation

1-a. In your AWS Cloud9 IDE environment, create a new file named vpc-subnets.yml under your CodeCommit repo and copy the template from the gist link below and paste to vpc-subnets.yml.

https://gist.github.com/carl-alarcon/80fe39e66845cc9e3f7e85a155058818

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image1.png)

1-b. Create an Amazon S3 bucket, which will host your CloudFormation templates, with the following command: 

```
aws s3 mb s3://cdmp.<YOUR_NAME> --region ap-southeast-1
```

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image2.png)

1-c. Upload vpc-subnets.yml to your newly created Amazon S3 bucket with the command: 

```
aws s3 cp vpc-subnets.yml s3://cdmp.<YOUR_NAME>
```
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image3.png)

1-d. Go to S3 in the management console, and navigate to the bucket containing the CloudFormation template. Select vpc-subnets.yml and copy the object URL.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image4.png)

1-e. Proceed to the CloudFormation console, click on create stack and select the option with new resources (standard). In step 1 select Amazon S3 URL and paste the URL in the textbox.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image5.png)

1-f. In step 2, enter the following details:

- Stack name: VPC-<YOUR_NAME>

- EnvironmentName: Development-<YOUR_NAME>

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image6.png)

1-g. Leave the rest of the configurations as default and create the vpc stack.

## 2. Create an Amazon EC2 Instance in AWS CloudFormation

2-a. In your AWS Cloud9 IDE environment, create a new file ec2.yml. Copy the template from the gist link below and paste to ec2.yml.

https://gist.github.com/carl-alarcon/780a488e9086aff3d32142c65895b0bb

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image7.png)

2-b. Upload ec2.yml to the Amazon S3 bucket containing your CloudFormation templates.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image8.png)

2-c. Following steps 1-d to 1-e, get the template URL from S3 and paste it to the CloudFormation Stack creation step 1. For step 2, enter the following details:

- Stack Name: EC2-< YOUR-NAME >

- CloudFormationVPC: < NAME-OF-YOUR-VPC-STACK >

- KeyPair: < NAME-OF-YOUR-KEYPAIR >

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+5/image9.png)

2-d. Leave the rest of the configurations as default and create the ec2 stack.



----------


## Challenge: Create your Amazon RDS Instance
You are challenged to create an Amazon RDS t2.micro instance on your own. Use the following reference documentation to create an Amazon RDS t2.micro instance.

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html

Please copy the JSON to the textbox below and supply the necessary information.

```
{
 "ec2_cloudformation_template_url": "",
 "cloudformation_s3_bucket_arn": "",
 "rds_cloudformation_template_url": ""
}
```

Reminder: Please don’t forget to delete your stack. Thanks!
