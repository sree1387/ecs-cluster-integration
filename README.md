# ecs-cluster-integration
git interation with aws ECR and ECS for hosting website in aws

**GIT WORKFLOW DEPLOYMENT WITH AWS ECS**

🚀 Overview
This repository follows a GitHub Actions workflow to automate the deployment of a Dockerized application to Amazon ECS (EC2 Launch Type) upon every push to the main branch.

📁 **Step-by-Step Setup**

1.** Create Git Repository

Create a new Git repository to store the app source code.
Add a .github/workflows/deploy.yml file for defining the deployment workflow.

**==================================================================================================================**

3. Configure GitHub Secrets
4. 
Add the following secrets in the GitHub repository (under Settings > Secrets and variables):

Key	Description
AWS_ACCESS_KEY_ID	IAM User Access Key
AWS_SECRET_ACCESS_KEY	IAM User Secret Access Key
======================================================================================================================


3. Push Project Files
Push your website or app code to the remote repository.
==============**====================================================================**

5. Setup AWS Resources
✅ IAM Setup
Create an IAM user with the following permissions:

ECR Access

ECS Cluster Access

EC2 Full Access

CloudWatch Logs Access

Attach the following IAM roles:

ecsInstanceRole

AmazonEC2ContainerServiceforEC2Role

AmazonElasticFileSystemClientReadWriteAccess

AmazonEC2FullAccess

ecsTaskExecutionRole

AmazonECSTaskExecutionRolePolicy

AmazonECS_FullAccess
============================================================================================================================

5. Create ECR Repository
Create a new ECR repository for your project.

Note down:

ECR repository name

ECR repository URI (e.g., 123456789012.dkr.ecr.us-west-2.amazonaws.com/project-repo-name)

Use this info in deploy.yml.
===========================================================================================================================


6. ECS Task Definition
Create a Task Definition for the application:

Launch Type: EC2

OS: Linux

eg
=====
CPU: 2 vCPU

Memory: 4GB
**based on the website or docker container requirement**

Container Name: your-container-name

Container Image: ECR URI

Port: Your app port (e.g., 3000)

Task Role: ecsTaskExecutionRole
===============================================================================================================================================

7. Create ECS Cluster
Launch Type: EC2 (not serverless)

Set:

Cluster Name: your-cluster-name

Instance Type: t3.medium or similar

Volume size: 30GB+ recommended

SSH key pair (to access the EC2 instance)

Enable:

CloudWatch Logging

Use the same VPC, subnet, and security group used in Task Definition
=========================================================================================================================================

8. Create ECS Service
Inside the ECS Cluster:

Create a Service using the Task Definition

Launch Type: EC2

Desired Tasks: 1+

Note the Service Name and use it in deploy.yml

⚙️ GitHub Actions: deploy.yml
Create .github/workflows/deploy.yml:

deploy.yml
===========

name: Deploy to ECS on Push

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest

    steps:
    - name: Checkout source code
      uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2  # ✅ Replace with your region

    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push Docker image to ECR
      env:
        ECR_REGISTRY: 123456789012.dkr.ecr.us-west-2.amazonaws.com
        ECR_REPOSITORY: sample-ecr-repo
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Deploy to ECS
      env:
        CLUSTER_NAME: sample-ecs-cluster
        SERVICE_NAME: sample-ecs-service
      run: |
        aws ecs update-service \
          --cluster $CLUSTER_NAME \
          --service $SERVICE_NAME \
          --force-new-deployment

 **====================================================================================================================**  

 
✅ Verification Steps
After pushing to main, go to the Actions tab to see workflow logs.

Check ECS console:

Is the Service updated?

Is the Task running successfully?

Go to EC2 Dashboard → Locate instance → SSH into it

Check Docker container is running:

docker ps
Access the app in browser:
**http://<EC2_Instance_Public_IP>:<Port>**
**=============================================================================================================================**


🔚 Notes
**Make sure EC2 Security Group allows inbound traffic on the app port.

Always update .env and other sensitive configs securely.

Monitor logs using CloudWatch for troubleshooting.



