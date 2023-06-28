# Jenkins-CICD
This project will use AWS instances to host Jenkins, SonarQube and Nexus code repository.
We will create a Jenkins Pipeline to fetch the source code from GitHub,
Build an artifact and perform unit test and checkstyle test with Maven,
Perform Code Analysis with SonarQube using Quality Gates and Webhooks
and we will deliver the artifact to Nexus code repository.

## Pre-requisites:
AWS account


# STEPS

## First Step - Setting up Servers

Connect to your AWS account and Launch 3 instances as follows:
(use scripts from userdata folder in this repository)
### Jenkins Server
Ubuntu OS//t2.small
key-pair: jenkins-key.pem
Security group: JenkinsSG
rule 1 - allow ssh (port 22) from MyIP
rule 2 - allow port 8080 from anywhere
rule 3 - allow HTTP (port 80) from anywhere

user data - use jenkins-setup.sh file

### Nexus Server
CentOS7//t2.medium
Key-pair: NexusKey.pem
Security group: NexusSG
rule 1 - allow ssh (port 22) from MyIP
rule 2 - allow port 8081 from anywhere
rule 3 - allow all traffic from JenkinsSG

user data - use nexus-setup.sh file

### SonarQube Server
Ubuntu OS//t2.medium
Key-pair: SonarKey
Security group: SonarSG
rule 1 - allow ssh (port 22) from MyIP
rule 2 - allow port 9000 from anywhere
rule 3 - allow HTTP (port 80) from anywhere

user data - use sonarqube-setup.sh file

## Validation
1. Validate each server using ssh - systemctl status jenkins/sonarqube/nexus
2. Connect through the browser with Public IP and Port separated for each of the 3:
3. For Jenkins - Take credentials and install suggested plugins
4. For Nexus - Sign in - follow instructions for credentials and reset password. Enable Anonymous access
5. For SonarQube - Login with admin/admin

# Jenkins Pipeline
## Install plugins for Jenkins
Go to Jenkins Browser to the main page (Dashboard) and on the left bar menu choose Manage Jenkins.
Then go to plugins page and mark the "Available plugins". In the search bar look for the following plugins and mark them:
Nexus Artifact Uploader
SonnarQube Scanner
Build TimeStamp
Pipeline Maven integration
Pipeline Utility Steps

Install without restart

## Install Tools
Maven, JDK, SonarQube and SonarQube Scanner

## Configurations and Credentials
