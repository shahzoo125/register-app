pipeline {
  agent { label 'Jenkins-Agent' }
  tools {
    jdk 'jdk21'
    maven 'Maven'
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
           
         }       
}
