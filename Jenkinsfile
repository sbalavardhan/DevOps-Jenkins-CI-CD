pipeline{
    agent{
       label "Infraserver"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "devops-jenkins-ci-cd"
        RELEASE = "1.0.0"
        DOCKER_USER = "sbalavardhan"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }

        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

        }
        stage("Trivy Scan") {
            steps {
                script {
		   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image sbalavardhan/devops-jenkins-ci-cd:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }

        }

        // stage ('Cleanup Artifacts') {
        //     steps {
        //         script {
        //             sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
        //             sh "docker rmi ${IMAGE_NAME}:latest"
        //         }
        //     }
        // }
        //
        //
        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.hrithihanvi.top/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
                }
            }

        }
    }
        post {
            failure{
    script {
               withCredentials([string(credentialsId: 'SLACK-WEBHOOK', variable: 'SLACK-WEBHOOK')]) {
                 sh """
                     msg='{"text":"Job ${env.JOB_NAME} (${env.BUILD_NUMBER}) has failed."}'
                     curl -X POST -H 'Content-type: application/json' --data "${msg}" "${SLACK-WEBHOOK}"
                    """
                }
    }
}

success{
    script {
               withCredentials([string(credentialsId: 'SLACK-WEBHOOK', variable: 'SLACK-WEBHOOK')]) {
                 sh """
                     msg='{"text":"Job ${env.JOB_NAME} (${env.BUILD_NUMBER}) has succeded."}'
                     curl -X POST -H 'Content-type: application/json' --data "${msg}" "${SLACK-WEBHOOK}"
                    """
                }
    }
}
        // failure {
        //       slackSend channel: '#jenkins', color: 'danger', message: '${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failure', teamDomain: 'devops-cicd', tokenCredentialId: 'SLACK-WEBHOOK'
        //     }
        //  success {
        //        slackSend channel: '#jenkins', color: 'danger', message: '${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - successfull', teamDomain: 'devops-cicd', tokenCredentialId: 'SLACK-WEBHOOK'
        //   }      
    }
       
}