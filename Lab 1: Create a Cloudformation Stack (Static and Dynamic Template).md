# Lab 1: Create a CloudFormation Stack (Static and Dynamic Template)

## Overview

This lab activity demonstrates how to create CloudFormation stacks using the AWS CLI. Once you have finished the lab, you will commit the CloudFormation templates to a GitHub repository.

---

## Prerequisites

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured on your machine, EC2 instance, or via [AWS CloudShell](https://docs.aws.amazon.com/cloudshell/latest/userguide/).
- Permissions to create CloudFormation stacks, S3 buckets, and EC2 instances.
- Access to a text editor (e.g., VS Code, nano, vim).
- [GitHub account](https://github.com/) to create a repository.

---

## 1. Create a CloudFormation Stack

### 1-a. Create the Static CloudFormation Template

1. In your terminal, create a file named `static-cf.yml`.
2. Copy the template from:  
   [https://github.com/mikerayco/cf-templates-demo/blob/master/static-template.yml](https://github.com/mikerayco/cf-templates-demo/blob/master/static-template.yml)
3. Edit the S3 bucket name in the YAML file to make it unique by adding random characters.  
   Example:
   ```
   BucketName: your-stack-bucketname-xyz123
   ```
1-b. Create the Stack
   ```
   aws cloudformation create-stack --stack-name cf-demo --template-body file://static-cf.yml
   ```
1-c. Validate Stack Creation
Go to the CloudFormation Console.
Confirm that the cf-demo stack exists and created an S3 bucket and EC2 instance.

2. Attempt to Create a Second Stack with the Same Template
2-a. Run This Command
```
   aws cloudformation create-stack --stack-name cf-demo2 --template-body file://static-cf.yml
```
Expected Result:
```
You will receive an error because the S3 bucket name is not unique (S3 bucket names are globally unique).
```

4. Create a Stack Using a Dynamic CloudFormation Template

3-a. Create the Dynamic Template
```
Create a file named dynamic-cf.yml.
```
Copy the template from: https://github.com/mikerayco/cf-templates-demo/blob/master/dynamic-template.yml

3-b. Create a New Stack with the Dynamic Template

```
aws cloudformation create-stack --stack-name dynamic-demo --template-body file://dynamic-cf.yml
```

3-c. Verify Stack Creation
Go to the CloudFormation Console and check the dynamic-demo stack.

3-d. Create Another Stack with the Same Template
```
aws cloudformation create-stack --stack-name dynamic-demo2 --template-body file://dynamic-cf.yml
```
3-e. Confirm Success
Check the dynamic-demo2 stack in the CloudFormation Console.
The dynamic template allows multiple stacks by using unique resource names.

6. Clean Up
Delete All Stacks Before Completing the Lab:

You can delete stacks using the AWS console or the CLI:

```
aws cloudformation delete-stack --stack-name cf-demo
aws cloudformation delete-stack --stack-name cf-demo2
aws cloudformation delete-stack --stack-name dynamic-demo
aws cloudformation delete-stack --stack-name dynamic-demo2
```

5. Push Your Templates to GitHub
Create a new public repository in your GitHub account named:
```
cf-template-Lab1
```
Initialize Git in your working directory, add both templates, and push to GitHub:

```
git init
git remote add origin https://github.com/YOUR_GITHUB_USER/cf-template-Lab1.git
git add static-cf.yml dynamic-cf.yml
git commit -m "Add CloudFormation templates for Lab 1"
git push -u origin main
```
Submission
Copy and paste the following JSON in the submission box, replacing with your actual GitHub repository name:
```
{
  "github_repository_name": "cf-template-Lab1"
}
```
