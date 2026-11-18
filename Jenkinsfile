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
           
         }       
}
