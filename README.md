# Streamlining Kubernetes Deployments: CI/CD with GitHub Actions and Helm for EKS
## Architecture Diagram
![Architecture](https://github.com/visala123/Eks-app-repo/blob/cd5d9c4b141a4f05f88dc684094c650c6089ccd1/Architecture.png)

## Values.yaml
update the values.yaml repository: XXXXXXXXXXXX.dkr.ecr.ap-northeast-2.amazonaws.com/eks-app-repo   #ECR registry URI 

## Sonar Scanner

SonarScanner Setup: Create an organization in sonarcloud

Open the browser and navigate to www.sonarcloud.io, click the "+" at the top right corner and create a new organization -> create one manually

name: github-actions-cicd

key: github-actions-cicd -> Create

Click analyze new project

Organization: github-actions-cicd

display name: github-actions

Project key: github-actions -> Next

The new code for this project will be based on: Previous version=true

-> Create project

Copy the sonar token and paste it in notepad, click on information on the left, copy the values of organization, project key and create secrets in Github settings

SonarQube Scan: Analyzes code quality, test coverage, style violations, and sends the results to a configured SonarQube instance.

Secrets used:

SONAR_URL, SONAR_TOKEN, SONAR_ORGANIZATION, SONAR_PROJECT_KEY

## GitHub Actions CI/CD Workflow Explanation

This project uses a GitHub Actions workflow to implement a full CI/CD pipeline for building, testing, scanning, and deploying a Java-based application to an AWS EKS cluster using Helm.

Workflow File: .github/workflows/main.yml

Workflow Triggers:

Manually via workflow_dispatch

## Jobs Breakdown:

## 1. Testing Job
   
This job performs code checkout, runs unit tests, performs code quality checks, and executes a SonarQube scan.

Steps:

Checkout Code: Uses the latest commit.

Maven Test: Runs unit tests with mvn test.

Checkstyle: Enforces code formatting using mvn checkstyle:checkstyle.

Java Setup: Uses Java 17 as required by SonarScanner.


## 2. Build_and_publish Job
   
This job builds a Docker image and pushes it to Amazon ECR, then runs a vulnerability scan using Trivy.

Steps:

Checkout Code

Build & Push Docker Image: Uses appleboy/docker-ecr-action to push the image to AWS ECR.

Trivy Scan: Scans the image for security vulnerabilities.

Upload Scan Results: Uploads the Trivy SARIF file to GitHub's Security tab for visibility under "Code Scanning Alerts".

Secrets used:

AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, REGISTRY

## 3. DeployToEKS Job
   
This job deploys the Docker image to an AWS EKS cluster using Helm.

Steps:

Checkout Code

Configure AWS CLI: Using GitHub's AWS credentials action.

Generate kubeconfig: To connect to the correct EKS cluster.

Create ECR Secret: Kubernetes secret regcred is created for image pulls from ECR.

Helm Deploy: Uses Bitovi Helm GitHub Action to deploy the app to the cluster using the helm/githubactions-charts chart.

Values passed to Helm:

appimage: Docker image URI from ECR 

apptag: GitHub run number for versioning 

## Folder Structure (relevant parts) 

![Folder Structure](https://github.com/visala123/Eks-app-repo/blob/cd5d9c4b141a4f05f88dc684094c650c6089ccd1/Folder%20Structure.png)

## Required GitHub Secrets:

| Secret Name             | Description                    |
| ----------------------- | ------------------------------ |
| `AWS_ACCESS_KEY_ID`     | AWS access key for deployment  |
| `AWS_SECRET_ACCESS_KEY` | AWS secret access key          |
| `REGISTRY`              | ECR registry URI               |
| `SONAR_URL`             | SonarQube server URL           |
| `SONAR_TOKEN`           | SonarQube token for auth       |
| `SONAR_ORGANIZATION`    | SonarCloud organization name   |
| `SONAR_PROJECT_KEY`     | Project key in SonarCloud/Qube |

Note :For example this is the  ECR registry URI - XXXXXXXXXXXX.dkr.ecr.ap-northeast-2.amazonaws.com/eks-app-repo

 in the Secret REGISTRY you have to pass XXXXXXXXXXXX.dkr.ecr.ap-northeast-2.amazonaws.com

## To Expose the application 

Run:

kubectl get ingress vpro-ingress

Copy the LoadBalancer DNS address from the ADDRESS column and open it in a web browser to access the application.

## Login with

username: admin_vp

passwrd: admin_vp

## Summary

Unit-tested & analyzed Java code 

Secure image pushed to ECR 

Scan results uploaded to GitHub Security 

App deployed to EKS using Helm 
