pipeline{
    agent{
       label "Infraserver"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    stages{
       stage("cleanup workspace"){
            steps{
                cleanWs()
            }
       }
        stage("checkout from scm"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/sbalavardhan/DevOps-Jenkins-CI-CD.git'
            }
        }
        stage("build stage"){
            steps{
                sh "mvn clean package"
            }

        }
        stage("testing the application"){
            steps{
                sh "mvn test"
            }
        }
         stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }

        }
    }    
}