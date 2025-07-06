## Overview:

This activity will demonstrate how to deploy a simple Web Application using CodeCommit, CodeDeploy, and CodePipeline.

Pre-requisites:
- This activity will require a Cloud9 environment that runs on Ubuntu.



## 1. Create a CodeCommit Repository.

1-a. Navigate to the AWS CodeCommit console and create your repository.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image1.png)


1-b. Fire up your Cloud9 environment and create a folder (in this demo my folder name is demo-app). Once the folder is created download the sample application from this URL: 

https://docs.aws.amazon.com/codepipeline/latest/userguide/samples/SampleApp_Linux.zip into the recently created folder.

Your code structure should be similar to this:


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image2.png)






1-c. While in the demo-app folder (your folder name might be different) initialize git and push it to your CodeCommit repository.


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image3.png)
































## 2. Create a Web Server


2-a. Navigate to the AWS Console and create an EC2 instance with the following configuration

- AMI: Amazon Linux 2
- Instance Type: t2.micro
- Security Group: Allow HTTP (port 80) and SSH (port 22) to the public
- Subnet: Place the EC2 instance inside a Public Subnet
- Make sure to enable Auto-assign Public IP (You know what will happen if you don’t do this)
- Make sure you have a copy of the SSH keys on your machine


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image4.png)













## 3. Create an IAM role for the Web Server

3-a. We need to allow our EC2 instance to use the CodeDeploy service, to do this we need to create a role that allows EC2 access to Code Deploy. Let’s navigate to IAM and create a role.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image5.png)












3-b. Select AWS Service and EC2, and click next.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image6.png)













3-c. Search for AmazonEC2RoleforAWSCodeDeploy and select it, then click next, it’s up to you if you want to set tags.


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image7.png)













3-d. Set the name of your role and click create.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image8.png)






3-e. Once the role is created, navigate back to your EC2 instance and attach the role.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image9.png)























## 4. Install the CodeDeploy agent in the EC2 instance.
4-a. Connect to your EC2 instance via SSH, then update the OS by running: 

```
sudo yum update -y
```


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image10.png)













4-b. Install ruby by running: 

```
sudo yum install ruby -y
```

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image11.png)








4-b. Download the CodeDeploy agent by running: 

```
wget https://aws-codedeploy-ap-southeast-1.s3.ap-southeast-1.amazonaws.com/latest/install
```

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image12.png)









4-c. To run the executable installer we need to change the permission of the file first, run the command: 

```
chmod +x ./install
```

Then to run the installer run: 

```

sudo ./install auto
```

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image13.png)














## 5. Create an IAM role for the CodeDeploy service.

5-a. Navigate back to the IAM console and create a role.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image14.png)












5-b. Select AWS Service and CodeDeploy.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image15.png)


Scroll down and select Codedeploy and click next.


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image16.png)




5-c. AWSCodeDeployRole is automatically attached to the role, just click next and add tags if you want.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image17.png)











5-d. Name your role and create

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image18.png)















## 6. Configure CodeDeploy


6-a. Navigate to the CodeDeploy console and click create application.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image19.png)



6-b. Set the name of your application and select EC2/On-premises for the compute platform.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image20.png)



6-c. Click on create deployment group.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image21.png)

























6-d. Set the name of your deployment group, select the role we have created earlier, and for the deployment type just select in-place


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image22.png)














6-e. For the environment configuration select Amazon EC2 Instances and for the tag, select the name of your EC2 instance.

If your EC2 instance doesn’t have the tag Name, please add it.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image23.png)









6-e. Leave the Agent configuration as default, for the deployment settings, select CodeDeployDefault.OnceAtATime. Then for the load balancer, remove the check that enables load balancing

Click create deployment group.


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image24.png)


## 7. Create a CodePipeline

7-a. Navigate to CodePipeline and click on create pipeline.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image25.png)


7-b. Set the name of your pipeline and leave other settings as default.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image26.png)


7-c. Select your CodeCommit Repository and branch name, leave other settings as default.


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image27.png)







7-d. Skip the build stage, in this demo, we will deploy an app that does not require a build service.


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image28.png)











7-e. For the deploy stage, select your CodeDeploy and deployment group.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image29.png)

Click next.


















7-f. Briefly review your configuration and click create pipeline.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image30.png)








7-g. Once the pipeline is created, it will start deploying our application from the CodeCommit repository.




![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image31.png)











7-h. Once it’s all green grab the EC2 Public DNS and resolve it in a web browser.





![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image32.png)

























## 8. Modify the code

8-a. Go back to your Cloud9 environment and modify index.html
You can put anything there as long as it is HTML, if you don’t have anything in mind to put there you can instead copy and paste the code from the gist file: 

https://gist.github.com/mikerayco/0011f8e686709de389ce1554b1a73a20

Once you have made the changes, commit and push it to your CodeCommit Repository.


![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image33.png)







8-b. Once you have pushed your changes, CodePipeline will automatically deploy your changes to the Server.



![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image34.png)







8-c. Once CodePipeline is all green, open the website again in a web browser.

There’s a chance that the website was cached on the web browser, if that happens just add ?t=1 at the end of the URL.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CDMP/Session+1/Lab+7/image35.png)


And there! You have successfully created a working CodePipeline!


Well done!

Please copy and paste the JSON in the textbox below and supply the necessary information.


```
{
 "codecommit-repository-name":"",
 "codepipeline-name":""
}
```

Reminder: Please don’t forget to delete your EC2 instance, CodeDeploy, and CodePipeline. Thanks!
