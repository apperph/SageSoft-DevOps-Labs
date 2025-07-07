# Lab: Deploy a Simple Web Application Using CodeCommit, CodeDeploy, and CodePipeline

## Overview

This activity demonstrates how to deploy a simple web application using CodeCommit, CodeDeploy, and CodePipeline.

---

## Prerequisites

- A Linux, Mac, or Windows environment with [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and git installed.  
  You can use AWS CloudShell, an EC2 instance, or your local machine.
- An IAM user/role with permissions for CodeCommit, EC2, CodeDeploy, and CodePipeline.

---

## 1. Create a CodeCommit Repository

**1-a.** Go to the [AWS CodeCommit console](https://console.aws.amazon.com/codesuite/codecommit/repositories) and create a new repository.

---

**1-b.** In your terminal or development environment, create a folder for your app.

```
mkdir demo-app
cd demo-app
```

Download the sample app:

```
curl -O https://docs.aws.amazon.com/codepipeline/latest/userguide/samples/SampleApp_Linux.zip
unzip SampleApp_Linux.zip
rm SampleApp_Linux.zip
```

Your folder structure should look like:

.
├── appspec.yml
├── index.html
├── scripts/
...
structure

<br>

1-c. Initialize git and push the app to your CodeCommit repository.

```
git init
git remote add origin https://git-codecommit.<your-region>.amazonaws.com/v1/repos/<your-repo-name>
git add .
git commit -m "Initial commit"
git push origin main
```

(See Git credentials setup for CodeCommit if you’re new to CodeCommit.)

2. Create a Web Server
3. 
2-a. Launch an EC2 instance from the AWS EC2 console:

```
AMI: Amazon Linux 2
Instance Type: t2.micro
Security Group: Allow HTTP (port 80) and SSH (port 22) from anywhere or your IP
Subnet: Place the instance inside a Public Subnet
Auto-assign Public IP: Enabled
Download or select your EC2 key pair
```

3. Create an IAM Role for the Web Server

3-a. Go to IAM Roles and click "Create role".

```
Trusted entity: AWS service
Service: EC2
Permissions: Attach the AmazonEC2RoleforAWSCodeDeploy managed policy
Name: e.g., EC2CodeDeployRole
```

Attach this role to your running EC2 instance:

Go to EC2 instance details > Actions > Security > Modify IAM role.

4. Install the CodeDeploy Agent on the EC2 Instance
   
4-a. SSH into your EC2 instance:
```
ssh -i <keyfile.pem> ec2-user@<public-dns>
```

4-b. Update the OS & install dependencies:
```
sudo yum update -y
sudo yum install ruby -y
```

4-c. Download and install the CodeDeploy agent:

(WARNING: use the matching S3 bucket for your region!  See Docs)

```
wget https://aws-codedeploy-<your-region>.s3.<your-region>.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo systemctl start codedeploy-agent
sudo systemctl enable codedeploy-agent
```

Test with:
```
sudo systemctl status codedeploy-agent
```

5. Create an IAM Role for CodeDeploy
   
5-a. Go to IAM Roles and click "Create role".

```
Trusted entity: AWS service
Service: CodeDeploy
Permissions: Attach the AWSCodeDeployRole managed policy
Name: e.g., CodeDeployServiceRole
```

6. Configure CodeDeploy
   
6-a. Go to the CodeDeploy console.

6-b. Click "Create application", set a name (e.g., WebAppDemo), and set EC2/On-premises as the compute platform.

6-c. Click "Create deployment group".

Name it (e.g., WebAppDemoDG)
For Service Role, select the CodeDeploy service role you created earlier (CodeDeployServiceRole)
Deployment type: In-place

6-d. Target the EC2 instance by tag or name.

You may need to tag your EC2 instance via EC2 console (use tag key “Name” with a meaningful value).
Under Environment configuration, select Amazon EC2 Instances and add the tag of your instance.
6-e. Leave settings at default (or customize), turn OFF the load balancer.

Click "Create deployment group".

7. Create a CodePipeline
   
7-a. Go to the CodePipeline console and click "Create pipeline".

Name your pipeline
Source provider: CodeCommit
Select your repository and branch

7-b. Skip the build stage.

7-c. For the deploy provider, select CodeDeploy and your deployment group.

7-d. Review and create the pipeline.

The pipeline will pull code from your CodeCommit repo and deploy it to the EC2 instance via CodeDeploy.

8. Verify the Deployment

8-a. Once CodePipeline is “green” for all stages, get your EC2 instance’s Public DNS (IPv4).
```
Open http://<your-ec2-public-dns> in your browser.
```
You should see the sample web app.

9. Update the App and Test CI/CD
    
9-a. Edit index.html in your local project. Add any content or use the gist:
```
https://gist.github.com/mikerayco/0011f8e686709de389ce1554b1a73a20
```
9-b. Commit and push:
```
git add index.html
git commit -m "Updated index.html"
git push origin main
```

9-c. CodePipeline will detect the change and re-deploy!
Refresh your EC2 site to see the update.

10. Cleanup
Don’t forget to delete:

EC2 instance
CodeDeploy application and deployment group
CodePipeline pipeline
You can delete most resources quickly in their respective AWS consoles.

Submission
Copy and paste the following JSON and supply your information:
```
{
 "codecommit-repository-name": "",
 "codepipeline-name": ""
}
```
Congratulations! You have deployed a web app using CodeCommit, CodeDeploy, and CodePipeline!

