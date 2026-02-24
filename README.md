# Jenkins CI_CD Pipeline

## Getting started

The objective of this project is to design a **Continuous Integration (CI)** and **Continuous Deployment (CD)** pipeline for deploying a  [Spring Boot application](https://github.com/eazytraining/PayMyBuddy) on AWS. We will implement the necessary steps to ensure code quality and security while automating the deployment process. Here is the git workflow:
* On the main branch **(main)**, all stages must be executed, except the review deployment.

* On **other branches**, only the following stages must be executed:

    - Automated Tests
    - Code Quality Check
    - Build and Packaging


## Prerequisites

Recall that whether you are running Jenkins on a host machine or docker container, make sure that **docker** is installed. 

**1) SonarCloud**

Create a sonarcloud account and generate a token that will be used in Jenkins. 

**![Generating the sonar token ](./images/prerequisites/generate-sonar-token-jenkins.png)**

**2) Jenkins**

---
a) Install the following plugins:

* **Docker Pipeline**
* **SonarQube Scanner** (for static code analysis)
* **SSH Agent** (for deployment)
* **Slack Notification** (for notification)

**3) **Slack**

- Step 1: Create a channel where notifications will be sent 
- Step 2: Choose Jenkins from Apps to be installed and click on arrow **Go to app**

**![Add Jenkins](./images/prerequisites/add-jenkins-to-slack.png)**

**![Go to Jenkins App](./images/prerequisites/go-to-jenkins-app.png)**

- Step 3: Retrieve the **Team Subdomain** and **Integration Token Credential ID** to be added in Jenkins credentials and during the configuration. Then, test the connection to ensure that it works well

**![Slack in Jenkins](./images/prerequisites/slack-token.png)**

b) Add **dockerhub** and **sonarcloud** credentials under Global scope

**![Dockerhub credentials](./images/prerequisites/dockerhub-cred.png)**

**![Sonar token](./images/prerequisites/sonar-in-jenkins.png)**

c) Add **STAGING_SSH_KEY** and **PRODUCTION_SSH_KEY** credentials under Jenkins scope

**![SSH keys](./images/prerequisites/ssh-key.png)**

## **1) Automated testing**

We will execute unitary and integration tests.

**![Tests](./images/tests/tests_success.png)**

***

## **2) Code quality**

We want to perform a static analysis of the code using SonarCloud. First, make sure to add the sonar token (done in the prerequisites' section) in the global credentials.  

**![Sonarqube](./images/security/sonarqube_success.png)**

**![Sonarqube results](./images/security/sonarqube_results.png)**

## **3) Compilation and Packaging**

Now, we want to build the jar file, then build the docker image and push the latter into DockerHub.

**![Build](./images/docker/build_success.png)**

**![Dockerhub](./images/docker/dockerhub_image.png)**


## **4) Deployment and Validation in staging environment**

**![Stage deploy](./images/deployment/stage_deploy.png)**

## **5) Deployment and Validation in production environment**

**![Prod deploy](./images/deployment/deploy_prod.png)**



