pipeline {
  agent { label 'Jenkins-Agent' }
  tools {
    jdk 'jdk21'
    maven 'Maven'
  }
  environment {
    APP_NAME = "register-app-pipeline"
    DOCKER_USER = "shahzaib125"
    IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
    IMAGE_TAG = "1.0.${BUILD_NUMBER}"
  }

         stages{
           stage("Cleanup Workspace"){
             steps{
               cleanWs()  
             }
           }
           stage("Checkout from SCM"){
             steps{
               git branch: 'main', credentialsId: 'github' , url: 'https://github.com/shahzoo125/register-app'
             }
           }
           stage("Build Application"){
             steps{
               sh 'mvn clean package'
             }
           }
           stage("Test Application"){
             steps{
               sh 'mvn test'
             }
           }
           stage("Sonarqube Analysis"){
             steps{
               script{
                 withSonarQubeEnv(credentialsId: 'sonar-jenkin-token'){
                   sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar"
                 }
               }
             }
           }
           stage("Quality Gate"){
             steps{
               script{
                 waitForQualityGate abortPipeline: false , credentialsId: 'sonar-jenkin-token'
               }
             }
           }
           stage("Build & Push Docker Image") {
             steps {
               script {
                 docker.withRegistry('', 'jenkin-docker') {
                 docker_image = docker.build("${IMAGE_NAME}")
                 docker_image.push("${IMAGE_TAG}")
                 docker_image.push('latest')
                 }
               }
             }
           }
           stage("Trivy Scan") {
             steps {
               script {
                 sh """
                 docker run --rm \
                 -v /var/run/docker.sock:/var/run/docker.sock \
                 aquasec/trivy:latest image \
                 --no-progress \
                 --scanners vuln \
                 --severity HIGH,CRITICAL \
                 ${IMAGE_NAME}:${IMAGE_TAG}
                 """
               }
             }
           }
           stage("Cleanup Artifact") {
             steps {
               script {
                 sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}" 
                 sh "docker rmi ${IMAGE_NAME}:latest" 
               }
             }
           }
           stage("Trigger CD Pipeline") {
             steps {
               script {
                 withCredentials([string(credentialsId: 'JENKINS_API_TOKEN', variable: 'JENKINS_API_TOKEN')]) {
                   sh """
                   curl -v -k --user "shahzaib:${JENKINS_API_TOKEN}" \
                    -X POST \
                    -H 'cache-control: no-cache' \
                    -H 'content-type: application/x-www-form-urlencoded' \
                    --data 'IMG_TAG=${IMAGE_TAG}' \
                    'http://ec2-54-144-32-106.compute-1.amazonaws.com:8080/job/gitops-register-app/buildWithParameters?token=gitops-token'
                """
            }
        }
    }
}

                   
           
         }       
}
