# Jenkins-CICD
This project will use AWS instances to host Jenkins, SonarQube and Nexus code repository.
We will create a Jenkins Pipeline to fetch the source code from GitHub,
Build an artifact and perform unit test and checkstyle test with Maven,
Perform Code Analysis with SonarQube using Quality Gates and Webhooks
and we will deliver the artifact to Nexus code repository.

## Pre-requisites:
AWS account


## STEPS

### STEP 1 - Setting up Servers

Connect to your AWS account and Launch 3 instances as follows:
(use scripts from userdata folder in this repository)
##### A note regarding Security groups: 
##### > NEVER open ports to "anywhere" - it's here for the ease of use and practice only!
##### > always better using https with the proper ACM attached rather than http

#### Jenkins Server
Ubuntu OS//t2.small

key-pair: jenkins-key.pem

Security group: JenkinsSG

rule 1 - allow ssh (port 22) from MyIP

rule 2 - allow port 8080 from anywhere

rule 3 - allow HTTP (port 80) from anywhere

user data - use jenkins-setup.sh file

#### Nexus Server
CentOS7//t2.medium

Key-pair: NexusKey.pem

Security group: NexusSG

rule 1 - allow ssh (port 22) from MyIP

rule 2 - allow port 8081 from anywhere

rule 3 - allow all traffic from JenkinsSG

user data - use nexus-setup.sh file

#### SonarQube Server
Ubuntu OS//t2.medium

Key-pair: SonarKey

Security group: SonarSG

rule 1 - allow ssh (port 22) from MyIP

rule 2 - allow port 9000 from anywhere

rule 3 - allow HTTP (port 80) from anywhere

user data - use sonarqube-setup.sh file

#### Validation
1. Validate each server using ssh - systemctl status jenkins/sonarqube/nexus
2. Connect through the browser with Public IP and Port separated for each of the 3:
3. For Jenkins - Take credentials and install suggested plugins
4. For Nexus - Sign in - follow instructions for credentials and reset password. Enable Anonymous access
5. For SonarQube - Login with admin/admin

## Jenkins Pipeline
### STEP 2 - Install plugins for Jenkins
Go to Jenkins Browser to the main page (Dashboard) and on the left bar menu choose Manage Jenkins.

Then go to plugins page and mark the "Available plugins". In the search bar look for the following plugins and mark them:

Nexus Artifact Uploader

SonnarQube Scanner

Build TimeStamp

Pipeline Maven integration

Pipeline Utility Steps

Install without restart

### STEP 3 - Install Tools
Jenkins Dashboard > Manage Jenkins > Tools

JDK > Add JDK > Name: OracleJDK8 | JAVA_HOME: /usr/lib/jvm/java-1.8.0-openjdk-amd64

Maven > Add Maven > Name: MAVEN3 | Version: any 3.something

SonarQube Scanner > Add SonarQube Scanner > Name: sonar4.7

<SAVE>

### STEP 4 - Define SonarQube with Jenkins
Jenkins Dashboard > Manage Jenkins > System

SonarQube servers > (Enable) Environment variables | Add SonarQube > Name: sonar | Server URL: https://sonar-server public IP port 9000 > SAVE

To get token go to SonarQube browser and login with admin/admin. Click the A square on top right corner > My Account > Security (tab) | Generate Tokens > jenkins > Generate.

Copy the token and go back to jenkins browser

Jenkins Dashboard > Manage Jenkins > System > SonarQube servers > Server authentication token > Add - jenkins > Kind: Secret text | ID&Description: MySonarToken > Add

Choose MySonarToken > SAVE

### STEP 5 - Define Quality Gates in SonarQube
SonarQube Browser

Quality Gates (Top Navigation bar) > Create > Name: vprofile-QG > SAVE

Add Condition > On Overall Code | Quality Gate fails when: Bugs | is greater than: 100 > Add Condition

IN ORDER FOR THE QUALITY GATE TO WORK WE NEED TO INITIATE A JENKINS JOB AND BUILD, SO THE PROJECT WILL APPEAR IN SONARQUBE

Jenkins Browser > New Job > Name: PAAC | Pipeline > OK |> Scroll down to Pipeline script and copy the jenkins full pipeline script > SAVE > Build Now (The Build will fail at some point)

Back to SonarQube Browser > Projects (Top Nav-Bar) > Click on project name (vprofile) > Project settings > Quality Gate > vprofile-QG

Project settings > Webhooks > Create > Name: jenkins-ci-webhooks | URL: http://jenkins-server public IP : Port | (Secret - leave empty) > Create

### STEP 6 - Define Nexus Repository with Jenkins
Nexus Browser

Sign in - follow instructions to get initial password > user: admin | password: initial password > Next > set new password > Next > Enable Anonymous Access > Next > Finish
We click on the wheel for configuration > on left menu - Repositories > Create repository > Maven2 (Hosted) > Name: vprofile-repo > Create repository

Jenkins Browser

Dashboard > Manage Jenkins > Credentials > Stores scoped to Jenkins - System > Global credentials > Add Credentials > Kind: User name with password | Username/Password: | Id & Description: NexusLogin > Create

## STEP 7 - Jenkins Build



