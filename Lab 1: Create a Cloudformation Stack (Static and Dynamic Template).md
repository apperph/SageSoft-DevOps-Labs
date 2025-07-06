## Overview:

This lab activity will demonstrate how to create a CloudFormation stack using the AWS CLI. Once you have finished the lab, you need to commit the CloudFormation template to a CodeCommit repository

Prerequisite: 
- This activity will require a Cloud9 IDE running on either Ubuntu or Amazon Linux 2.


## 1. Create a CloudFormation stack.

1-a. Create a file named static-cf.yml, and copy the template from this URL: 

https://github.com/mikerayco/cf-templates-demo/blob/master/static-template.yml

Set the name of your bucket, add some random characters to the name of your bucket to make it unique.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image1.png)





1-b. Run the command: 

```
aws cloudformation create-stack --stack-name cf-demo --template-body file://static-cf.yml
```

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image2.png)


1-c. Navigate back to the CloudFormation console and check your stack.

It created an S3 bucket and EC2 instance.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image3.png)








## 2. Create a new stack using the same CloudFormation template

2-a. Navigate back to the Cloud9 IDE and create a new stack using the same template by running this command: 

```
aws cloudformation create-stack --stack-name cf-demo2 --template-body file://static-cf.yml
```

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image4.png)



You’ll see that there will be an error, the S3 namespace is universal and we cannot create a bucket name that is not unique.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image5.png)


Therefore the template we have used to create resources is considered static, although it works. We cannot reuse the template to create the same set of resources unless we rename it.




## 3. Create a stack using a dynamic CloudFormation template.


3-a. Navigate to your Cloud9 IDE and create a file named dynamic-cf.yml and paste the template from this url: 

https://github.com/mikerayco/cf-templates-demo/blob/master/dynamic-template.yml


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image6.png)



3-b. Create a stack by running this command: 

```
aws cloudformation create-stack --stack-name dynamic-demo --template-body file://dynamic-cf.yml
```

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image7.png)

3-c. Navigate to the CloudFormation console and check the stack. The stack was successfully created!

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image8.png)



3-d. Create another stack using the same template by running this command: 

```
aws cloudformation create-stack --stack-name dynamic-demo2 --template-body file://dynamic-cf.yml
```
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image9.png)






3-e. Navigate back to the CloudFormation console and check the recently created stack.


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+2/image10.png)


----------





Well done!!!

Please delete the stacks before ending this lab activity. Once you have deleted the stacks, create a repository in your github and use the naming convention: cf-template-<name>

Then push the CloudFormation templates in the repository.

Please copy and paste the JSON in the textbox below and supply the necessary information.


```
{
 "github_repository_name":""
}
```

