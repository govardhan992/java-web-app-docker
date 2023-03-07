pipeline {
    agent any
    tools {
          maven 'maven 3.9.0' 
    }
    stages {
        stage('check out'){
            steps {
                git 'https://github.com/govardhan992/java-web-app-docker.git'
            }
        }
        stage('build'){
            steps {
                sh "mvn clean package"
            }
        }
        stage('SonarQube Analysis'){
            steps{
                   withSonarQubeEnv('sonarqubeserver') {
                        sh 'mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=maven \
                        -Dsonar.host.url=http://54.172.72.238:9000 \
                        -Dsonar.login=1b52a17114761a2ecb2986b3bd52c95006cc5c66'
                    }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 4, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        } 
        stage('build the docker image'){
            steps {
                sh "docker build -t govardhanr992/java-web-app:${BUILD_NUMBER} ."
            }
        }
        stage("push docker image"){
            steps {
               withCredentials([string(credentialsId: 'docker_hub_pwd', variable: 'docker_hub_pwd')]) {
                
               sh "docker login -u govardhanr992 -p ${docker_hub_pwd}"
     }
                sh "docker push govardhanr992/java-web-app:${BUILD_NUMBER}"
            }
        }
        stage("deploy_app"){
            steps {
              script {

                 
                 sshagent(['deploy_application']) {
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.6.197 docker pull govardhanr992/java-web-app:${BUILD_NUMBER} '
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.6.197 docker rm -f webserver || true'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.6.197 docker run -d -p 8080:8080 --name webserver govardhanr992/java-web-app:${BUILD_NUMBER}'               
            }
                
              }
                }
                 }
                  }
                   }

