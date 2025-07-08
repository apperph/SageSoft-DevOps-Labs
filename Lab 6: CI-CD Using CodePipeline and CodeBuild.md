# Lab: Automate Containerization and CI/CD to ECR with CodePipeline (using Amazon EC2)

## Overview
In this lab, you will automate the containerization of a Java Spring Boot app and push the container image to Amazon ECR, orchestrated via AWS CodePipeline.

---

## Pre-requisites

1. **Amazon EC2 Instance** running **Amazon Linux 2023 AMI**
    - Instance type: t3.small or larger (with at least 2 vCPUs and 2 GiB RAM for Docker builds)
    - Assign an IAM profile with ECR and CodeBuild permissions, or configure AWS CLI with a user granting these permissions

2. **Verify EC2 Instance Setup**
    - Ensure you can SSH into your EC2 instance
    - You have `sudo` access

---

## 1. Build the Container Image

### 1-a. Clone the Repository

SSH into your EC2 instance and run:

```bash
git clone https://github.com/carl-alarcon/java-container-demo.git
cd java-container-demo
```
1-b. Install Java, Maven, and Docker
# Update packages
sudo dnf update -y

# Install Java 8 and Maven
sudo dnf install -y java-1.8.0-openjdk-devel maven

# Install Docker
sudo dnf install -y docker
sudo systemctl enable --now docker

# Add your user to the docker group for non-root use
sudo usermod -aG docker $USER
# Refresh group membership:
newgrp docker
Verify installations:

java -version
mvn -version
docker --version
1-c. Test Spring Boot App Locally
mvn spring-boot:run
Open a second SSH session and run:

curl http://localhost:8080/api/hello
You should see a hello message.
Stop the app (Ctrl+C in the first window) when finished.

1-d. Package the Application
mvn package
1-e. Create a Dockerfile
Create a file named Dockerfile:

FROM openjdk:8-jdk-alpine
COPY target/demo-0.0.1-SNAPSHOT.jar /
CMD ["java", "-jar", "/demo-0.0.1-SNAPSHOT.jar"]
1-f. Create an ECR Repository
Go to Amazon ECR > Create repository
Choose Private
Name: e.g., java-container-demo
Click Create repository
1-g. Authenticate Docker to ECR
On your EC2 instance:

aws ecr get-login-password --region <region> | \
docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<region>.amazonaws.com
Replace <region> and <AWS_ACCOUNT_ID> as appropriate.

1-h. Build and Push the Docker Image
export ECR_URI=<AWS_ACCOUNT_ID>.dkr.ecr.<region>.amazonaws.com/java-container-demo
docker build -t $ECR_URI:latest .
docker push $ECR_URI:latest
2. Build Stage
2-a. Create a GitHub Repository
Create a new GitHub repository, e.g., java-container-demo.

2-b. Push Your Code to GitHub
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
2-c. Create buildspec.yml at Repo Root
Create a file named buildspec.yml:

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
Commit and push:

git add buildspec.yml
git commit -m "Add buildspec for CodeBuild"
git push
2-d. Set Up AWS CodeBuild Project
In AWS CodeBuild, click Create build project
Source: Connect to your GitHub repository (GitHub App recommended)
Environment: Managed image, Ubuntu, standard, privileged enabled
Service Role: New or existing
Add environment variables:
IMAGE_REPO_NAME = java-container-demo
IMAGE_TAG = latest
AWS_ACCOUNT_ID = your account ID
2-e. Add IAM Policy to CodeBuild Role
Go to IAM > Roles, search for the build service role
Attach AmazonEC2ContainerRegistryPowerUser policy
2-f. Start a Build to Test
Use Start build in the CodeBuild console. Confirm that it pushes the image to ECR.

3. Create a Pipeline
3-a. Use CodePipeline
Source: Your GitHub repo
Build: Select the CodeBuild project you created
Deploy: (Skip, or add a test step)
Name your pipeline
Finish pipeline creation.
Commit & push a test change to your repo. Verify the pipeline triggers and pushes an image to ECR.

Copy the following JSON, fill in, and submit as instructed:
{
  "pipeline_name": "",
  "codebuild_project_name": "",
  "GitHub_repo_link": "",
  "ECR_Repo_name": ""
}
Cleanup
Delete CodeBuild project
Delete CodePipeline
Delete ECR repo
Terminate your EC2 instance
