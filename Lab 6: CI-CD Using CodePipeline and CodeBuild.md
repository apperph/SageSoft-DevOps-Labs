## Overview
In this lab, you will automate the containerization and pushing of the image to ECR with the help of CodePipeline.

## Pre-requisite

1. Cloud9 Amazon Linux 2023 environment
2. Verify that docker is installed on the Cloud9 environment

## 1. Build the container image
1-a. Clone the repository into the Cloud9 environment that you provisioned: [GitHub repository](https://github.com/carl-alarcon/java-container-demo.git)

```
git clone https://github.com/carl-alarcon/java-container-demo.git
```
The repository contains an application with web and actuator as dependencies generated from https://start.spring.io/ 

1-b. Install Maven on your cloud9 environment following the instructions below

```
sudo wget https://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
sudo yum install -y apache-maven
sudo yum install java-1.8.0-devel
sudo /usr/sbin/alternatives --config java
sudo /usr/sbin/alternatives --config javac
```
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image2-0.png)

1-c. Now, go to your application folder and execute the following Maven command to start your Spring Boot application. This will allow you to test the application before placing it inside a container.

```
mvn spring-boot:run
```
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image2-1.png)

You can test your application by opening a new terminal on the cloud9 environment and running curl on http://localhost:8080/api/hello

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image2-2.png)

1-d. Package the application using Maven.

```
mvn package
```
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image2-3.png)

1-e. Create a Dockerfile to your folder with the contents below.

```
# Our base image that contains OpenJDK
FROM openjdk 

# Add the fatjar in the image
COPY target/demo-0.0.1-SNAPSHOT.jar / 

# Default command
CMD java -jar /demo-0.0.1-SNAPSHOT.jar
```
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image2-4.png)

1-f. Go to the ECR management console and click on Create Repository. Choose private on the visibility settings and enter your selected repository name and click on Create Repository.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image2-4a.png)

1-g. On the ECR console, select your repository and click on View push commands
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/DevOps/Lab0/image0-5.png)

1-h Authenticate your cloud9 docker client to your registry by running the first command shown on the first part of the pop-out window.
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/DevOps/Lab0/image0-6.png)

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/DevOps/Lab0/image0-7.png)

1-i. Build your Docker image with the command: docker build -t [ECR URI]:latest .

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/DevOps/Lab0/image0-8.png)

1-j. Push the image to ECR using this command: docker push image_repository/name_of_image:tag
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/DevOps/Lab0/image0-10.png)


## 2 Build Stage
2-a. Create a repository for the application on GitHub.

2-b. Push application files of the Java app from the previous steps to the GitHub repo.

2-c. Create the buildspec.yml on the root directory with the contents below. This buildspec is used for automating dockerization. It contains certain commands necessary to containerize our application. Push the buildspec.yml file to the GitHub repo as well.

```
version: 0.2

phases:
  install:
    commands:    
      - mvn install
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - echo Build started on `date`
      - mvn package
      - echo Building the Docker image...          
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG      
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
```

2-d. Now, we can create a CodeBuild Project.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1a.png)

2-e. Configure your GitHub repository as source using personal access token.
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1b.png)

2-f. Enter the following compute configuration under the environment section. 

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1c.png)

2-g. Select New Service Role then we will attach AmazonEC2ContainerRegistryPowerUser policy later. This role permits CodeBuild to upload images to ECR.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1d.png)

2-h. Click Additional Configuration below the role name. Here we will set environment variables that will be referenced by the buildspec file. We can use the same ECR registry we used earlier.

> Note: Don’t forget to tick the Privileged config. This will run docker daemon automatically.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/DevOps/Lab1/image1-2.png)

2-i. Under Buildspec, select “Use a buildspec file”. CodeBuild will automatically look for a file named buildspec.yml on the root directory. Otherwise, you can specify the name of your build spec file accordingly. You can also insert build commands directly using the console.
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/DevOps/Lab1/image1-4.png)

2-j. Select “Create build project”

2-k. Let’s attach AmazonEC2ContainerRegistryPowerUser to the CodeBuild service role we created. Select the build project you just created. On the configuration, you will find the Service role. Then click the link to edit the role.
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1e.png)

2-i. You can select “Start build” to test whether the build will push through and troubleshoot in advance before integrating it into the CodePipeline.
![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1e.png)

> Note: Once build is successful, we can create our CodePipeline to orchestrate everything. For every push to the GitHub repository, your pipeline will automatically trigger the pipeline to release the changes and begin containerizing your application.

## 3. Create a Pipeline
3-a. Go to CodePipeline, and click “Create pipeline”. Configure pipeline settings. See below for details.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1f.png)

3-b. Click Next, select the repository of your java application under the Source stage. See below photo for your reference.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1g.png)

3-c. Click Next to select the build project you created using the Build stage option. Then refer to the photo below for more details.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1h.png)

3-d. Click Next, then skip the Deploy stage and select “Create Pipeline”.

3-e. You now have created a simple CI/CD pipeline that builds images automatically for every “push” change on your code repository. CodePipeline console will indicate every state of pipeline stages.

![](https://sb-next-prod-image-bucket.s3.ap-southeast-1.amazonaws.com/public/CNMP/ARCH1008-2/1008image1-1i.png)

Congratulations, you have completed the lab!

Please copy the JSON to the textbox on the left side and supply the necessary information.

```
{
 "pipeline_name": "",
 "codebuild_project_name": "",
 "GitHub_repo_link":"",
 "ECR_Repo_name":""
}
```

You can now delete the CodeBuild project, CodePipeline, and ECR you have created.
