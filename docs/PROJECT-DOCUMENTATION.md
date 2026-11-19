# Register App --- DevOps CI/CD & GitOps Project

## 1. Project Overview

This project demonstrates an end-to-end DevOps workflow for building,
testing, scanning, containerizing, and deploying a Java web application
on Amazon EKS.

The application source code is maintained in the **register-app**
repository. Jenkins automates the CI process, SonarQube performs code
quality analysis, Trivy scans the Docker image, and the CD pipeline
updates the Kubernetes deployment configuration in a separate GitOps
repository. Argo CD watches that repository and synchronizes the
application to the EKS cluster.

### Repositories

-   **Application:** https://github.com/shahzoo125/register-app
-   **GitOps manifests:**
    https://github.com/shahzoo125/gitops-register-app
-   **Docker Hub image:**
    https://hub.docker.com/r/shahzaib125/register-app-pipeline

## 2. Technologies Used

-   **Source control:** Git, GitHub
-   **CI/CD:** Jenkins
-   **Build:** Java 21, Maven
-   **Code quality:** SonarQube
-   **Containerization:** Docker
-   **Image security scanning:** Trivy
-   **Container registry:** Docker Hub
-   **Container orchestration:** Kubernetes, Amazon EKS
-   **GitOps continuous delivery:** Argo CD
-   **Cloud platform:** AWS
-   **Application server:** Apache Tomcat

## 3. Architecture

The project uses two Jenkins pipelines and a separate GitOps repository.

1.  A developer pushes application code to the application GitHub
    repository.
2.  The Jenkins CI pipeline checks out the source code and builds the
    application with Maven.
3.  Maven tests are executed.
4.  SonarQube analyzes the code and the pipeline checks the Quality
    Gate.
5.  Jenkins builds a Docker image and pushes the versioned image and
    `latest` tag to Docker Hub.
6.  Trivy scans the built image for HIGH and CRITICAL vulnerabilities.
7.  The CI pipeline triggers the CD Jenkins job and passes the new image
    tag.
8.  The CD pipeline updates the image version in the GitOps repository.
9.  Argo CD detects the GitOps repository change and synchronizes the
    Kubernetes resources on Amazon EKS.
10. The application is exposed through a Kubernetes LoadBalancer
    service.

## 4. AWS Infrastructure
<img width="1914" height="1023" alt="running instances" src="https://github.com/user-attachments/assets/4d739883-45b9-40f0-bcba-cb15b0326520" />


<img width="1920" height="1030" alt="Sonarqube" src="https://github.com/user-attachments/assets/d7f38864-4294-403f-a3a1-708fd2d73940" />

The project uses separate EC2 instances for the required DevOps tools
and administration tasks.

  Component          Purpose
  ------------------ ----------------------------------------------
  Jenkins Master     Hosts Jenkins and manages pipeline execution
  Jenkins Agent      Runs the CI pipeline stages
  SonarQube Server   Performs code quality analysis
  Bootstrap Server   Used to configure and manage the EKS cluster
  Amazon EKS         Hosts the Kubernetes application workloads

### Jenkins Master
<img width="1900" height="1030" alt="pipeline-jobs" src="https://github.com/user-attachments/assets/af97787b-ffbb-49f8-bb4f-30378db20e86" />



Jenkins was installed on an Ubuntu EC2 instance.

Main setup tasks:

-   Updated the Ubuntu packages.
-   Set the server hostname.
-   Installed Java 21.
-   Added the Jenkins package repository and installed Jenkins.
-   Started and enabled the Jenkins service.
-   Allowed access to Jenkins through port `8080`.
-   Configured the Jenkins controller to use zero executors because
    pipeline work is performed by the Jenkins Agent.

Open Jenkins at:

``` text
http://<JENKINS_SERVER_PUBLIC_IP>:8080
```

### Jenkins Agent

A separate Ubuntu EC2 instance was configured as the Jenkins Agent.

Main setup tasks:

-   Installed Java 21 and Docker.
-   Added the Jenkins user to the Docker group so pipeline jobs could
    run Docker commands.
-   Configured SSH access between the Jenkins controller and agent using
    an SSH key pair.
-   Added the public key to the agent user's `authorized_keys`.
-   Added the agent in Jenkins and configured its connection details and
    label.

The pipeline uses the agent label:

``` groovy
agent { label 'Jenkins-Agent' }
```

### SonarQube Server
<img width="1920" height="1030" alt="Sonarqube" src="https://github.com/user-attachments/assets/4b52b12d-c0fc-4538-ac9a-4eb5c8633e1c" />


SonarQube was installed on a separate EC2 instance. PostgreSQL was
configured as its database.

Main setup tasks:

-   Created the SonarQube EC2 instance.
-   Installed and configured PostgreSQL.
-   Created the SonarQube database and database user.
-   Installed SonarQube under `/opt/sonarqube`.
-   Configured the database connection in `sonar.properties`.
-   Configured the SonarQube service.
-   Started the service and accessed the SonarQube web interface on port
    `9000`.
-   Integrated SonarQube with Jenkins using the required Jenkins plugin,
    server configuration, and authentication token.
-   Configured the SonarQube webhook required for the Jenkins Quality
    Gate check.

The database connection uses the following format. Replace the
placeholders with the values configured on your server:

``` properties
sonar.jdbc.username=<DATABASE_USERNAME>
sonar.jdbc.password=<DATABASE_PASSWORD>
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

SonarQube URL:

``` text
http://<SONARQUBE_SERVER_PUBLIC_IP>:9000
```

## 5. Jenkins CI Pipeline
<img width="1891" height="967" alt="jenkins-Ci" src="https://github.com/user-attachments/assets/fa01162e-efb3-40fd-9277-df791304aed5" />



The CI pipeline is stored in the application repository in a
`Jenkinsfile`. It runs on the Jenkins Agent.

### Pipeline stages

  -----------------------------------------------------------------------
  Stage                               Purpose
  ----------------------------------- -----------------------------------
  Cleanup Workspace                   Removes files from the previous
                                      build

  Checkout from SCM                   Checks out the application source
                                      from GitHub

  Build Application                   Builds the Java application using
                                      Maven

  Test Application                    Runs Maven tests

  SonarQube Analysis                  Sends the project for code quality
                                      analysis

  Quality Gate                        Checks the SonarQube Quality Gate
                                      result

  Build & Push Docker Image           Builds the Docker image and pushes
                                      tags to Docker Hub

  Trivy Scan                          Scans the Docker image for HIGH and
                                      CRITICAL vulnerabilities

  Cleanup Artifact                    Removes the locally built image
                                      tags from the Jenkins Agent

  Trigger CD Pipeline                 Starts the CD Jenkins job and
                                      passes the image tag
  -----------------------------------------------------------------------

### Application build

Maven is used to build and package the application:

``` bash
mvn clean package
```

Tests are executed with:

``` bash
mvn test
```

### SonarQube analysis

The Maven SonarQube scanner is used to submit the analysis:

``` bash
mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar
```

Jenkins waits for the Quality Gate result before continuing.

### Docker image
<img width="1893" height="980" alt="dockerhub" src="https://github.com/user-attachments/assets/bb7e1ef6-6e65-41e5-b854-67e95538a2e1" />



The image is built and pushed to Docker Hub using the project image
name:

``` text
shahzaib125/register-app-pipeline
```

The pipeline publishes a build-specific tag in the format:

``` text
1.0.<BUILD_NUMBER>
```

It also pushes the `latest` tag.

### Trivy image scan

Trivy scans the versioned image for HIGH and CRITICAL vulnerabilities.
The scan is performed against the image built by the pipeline.

### Local image cleanup

After the scan, the pipeline removes the versioned and `latest` image
tags from the Jenkins Agent to clean up local Docker storage.

### Triggering the CD pipeline

<img width="1872" height="962" alt="gitops-register CD" src="https://github.com/user-attachments/assets/b53a05c9-e5b0-427c-8e30-941d80c0cd17" />



After the CI stages complete, Jenkins triggers the GitOps CD job and
passes the newly created image tag as the `IMG_TAG` build parameter.

## 6. Docker Configuration

The application is packaged as a WAR file and deployed on Apache Tomcat.

The project's Dockerfile:

``` dockerfile
FROM tomcat:latest
RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
COPY webapp/target/*.war /usr/local/tomcat/webapps/
```

The Docker image contains Tomcat and the application WAR file.

## 7. Amazon EKS Setup

A Bootstrap EC2 instance was used to manage the Kubernetes cluster.

### Bootstrap server tools

The server was prepared with the tools required to work with AWS and
Kubernetes:

-   AWS CLI
-   `kubectl`
-   `eksctl`

An IAM role was attached to the Bootstrap server to provide the required
AWS permissions.

### EKS cluster creation

The cluster was created in `us-east-1` using `eksctl` with a managed
node group.

The cluster configuration used was:

``` bash
eksctl create cluster \
  --name my-eks-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.small \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

This creates the EKS control plane, which is managed by AWS, and the
managed worker node group in the AWS account.

### Verify the cluster

``` bash
kubectl get nodes
kubectl get pods -A
```


## 8. Argo CD Setup

Argo CD was installed in the EKS cluster to implement GitOps-based
continuous delivery.

Main setup tasks:

-   Created the Argo CD namespace.
-   Installed Argo CD using its Kubernetes installation manifests.
-   Exposed the Argo CD server using a Kubernetes LoadBalancer service.
-   Retrieved the initial administrator password and logged in to the
    Argo CD CLI.
-   Registered the EKS cluster with Argo CD.
-   Connected the GitOps GitHub repository.
-   Created the Argo CD application using the Kubernetes manifests
    stored in the GitOps repository.

Argo CD continuously monitors the configured Git repository and
synchronizes the desired Kubernetes state with the cluster.




## 9. Jenkins CD Pipeline and GitOps Workflow

The CD pipeline is configured as a separate Jenkins job. It receives the
`IMG_TAG` parameter from the CI pipeline.

Its role in the deployment workflow is to update the application image
version in the GitOps repository. Argo CD then detects the repository
change and synchronizes the updated Kubernetes configuration to Amazon
EKS.

The GitOps repository is:

``` text
https://github.com/shahzoo125/gitops-register-app
```

This separates application source code from Kubernetes deployment
configuration.

## 10. Application Access

The application is exposed through a Kubernetes LoadBalancer service.

Use the LoadBalancer DNS name with the Tomcat application context path:

``` text
http://<LOADBALANCER_DNS>:8080/webapp/
```

The Tomcat default page may appear at the root path `/`. The application
itself is available under `/webapp/`.

<img width="1920" height="1080" alt="Screenshot (59)" src="https://github.com/user-attachments/assets/75b66e4a-88d4-4032-83f9-48f9dde36581" />





## 11. End-to-End Deployment Flow

``` text
Developer
   |
   v
Application GitHub Repository
   |
   v
Jenkins CI Pipeline (Jenkins Agent)
   |
   +--> Maven Build
   |
   +--> Maven Tests
   |
   +--> SonarQube Analysis + Quality Gate
   |
   +--> Docker Build and Push
   |
   +--> Trivy Image Scan
   |
   v
Trigger Jenkins CD Pipeline (IMG_TAG)
   |
   v
GitOps GitHub Repository
   |
   v
Argo CD
   |
   v
Amazon EKS
   |
   v
Kubernetes LoadBalancer
   |
   v
Web Application
```

## 12. Screenshots

Add your project screenshots in a `screenshots` folder and update the
image paths below to match the actual filenames.

  ----------------------------------------------------------------------------
  Screenshot                               What it demonstrates
  ---------------------------------------- -----------------------------------
  `<img width="1914" height="1023" alt="running instances" src="https://github.com/user-attachments/assets/604d7f93-b4d8-465b-8b6c-e736a9712f05" />`  AWS EC2 infrastructure

  `screenshots/jenkins-ci-success.png`     Successful Jenkins CI pipeline

  `screenshots/sonarqube-analysis.png`     SonarQube project analysis

  `screenshots/dockerhub-image.png`        Published Docker image

  `screenshots/eks-nodes.png`              EKS worker nodes

  `screenshots/kubernetes-resources.png`   Kubernetes pods and resources

  `screenshots/argocd-application.png`     Argo CD application synchronization

  `screenshots/jenkins-cd-success.png`     Jenkins CD pipeline

  `screenshots/application-ui.png`         Deployed application interface
  ----------------------------------------------------------------------------

Example Markdown to display a screenshot:

``` markdown
![Successful Jenkins CI Pipeline](screenshots/jenkins-ci-success.png)
```

## 13. Security Notes

-   Do not commit passwords, personal access tokens, API tokens, SSH
    private keys, or AWS credentials to GitHub.
-   Store Jenkins credentials in the Jenkins Credentials Manager.
-   Use IAM roles for AWS access from EC2 instances instead of storing
    long-term AWS access keys on the server.
-   Keep secret values out of screenshots and project documentation.
-   Replace example placeholders with non-sensitive information only
    when necessary.

## 14. Project Outcome

This project demonstrates an automated DevOps workflow for a Java web
application, including:

-   Source code management with GitHub.
-   Automated Maven build and testing through Jenkins.
-   Static code analysis and Quality Gate validation with SonarQube.
-   Docker image creation and publishing to Docker Hub.
-   Vulnerability scanning with Trivy.
-   Kubernetes deployment on Amazon EKS.
-   GitOps-based delivery and synchronization using Argo CD.
