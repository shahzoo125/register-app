pipeline {
  agent { label : 'Jenkins-Agent' }
  tools {
    jdk 'jdk21'
    maven 'Maven'
  }
         stages{
           stage("Cleanup Workspace"){
             steps{
               cleanws()  
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
           
         }       
}
