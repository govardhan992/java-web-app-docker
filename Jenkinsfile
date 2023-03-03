pipeline {
    agent any
    tools {
          maven 'maven-3.9.0' 
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
                   withSonarQubeEnv('Sonarqube-8. 9.2') {
                        sh 'mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=maven \
                        -Dsonar.host.url=http://18.116.32.221:9000   \
                        -Dsonar.login=c8c9d5325de5d8eb33d201d9e1f572f3163324d8'
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
                sh "docker build -t govardhanr992/javaapp:${BUILD_NUMBER} ."
            }
        }
        stage("push docker image"){
            steps {
                withCredentials([string(credentialsId: 'Dockerhub_pwd', variable: 'Dockerhub_pwd')]) {
                
               sh "docker login -u govardhanr992 -p ${Dockerhub_pwd}"
     }
                sh "docker push govardhanr992/javaapp:${BUILD_NUMBER}"
            }
        }
        stage("deploy_app"){
            steps {
              script {

                 def USER_INPUT = input(
                    message: 'User input required - Do you want to proceed?',
                    parameters: [
                            [$class: 'ChoiceParameterDefinition',
                             choices: ['no','yes'].join('\n'),
                             name: 'input',
                             description: 'Menu - select box option']
                    ])
                    echo "The answer is: ${USER_INPUT}"
                    if( "${USER_INPUT}" == "yes"){
                sshagent(['deploy_container']) {
                 
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.27.31 docker pull govardhanr992/javaapp:${BUILD_NUMBER}'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.27.31 docker rm -f webserver || true'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.27.31 docker run -d -p 8080:8080 --name webserver govardhanr992/webserver:${BUILD_NUMBER}'
                }
                   
            }
                else {
                   echo "Skipping deployment"
                }
         }
            }
        }
    }
    }

