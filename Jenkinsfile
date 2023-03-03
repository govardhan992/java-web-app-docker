pipeline {
    agent any
    tools {
          maven 'Maven 3.0.5' 
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
                   withSonarQubeEnv('Sonarqubeserver') {
                        sh 'mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=maven-project \
                        -Dsonar.host.url=http://3.14.248.244:9000 \
                        -Dsonar.login=af8a2ecfe3ba8a7ccc6a500e938921767e93c68d'
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
                sh "docker build -t govardhanr992/java-web:${BUILD_NUMBER} ."
            }
        }
        stage("push docker image"){
            steps {
               withCredentials([string(credentialsId: 'docker_hub_pwd', variable: 'docker_hub_pwd')]) {
                
               sh "docker login -u govardhanr992 -p ${docker_hub_pwd}"
     }
                sh "docker push govardhanr992/java-web:${BUILD_NUMBER}"
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
                sshagent(['deploy_app']) {
                 
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@10.0.1.24 docker pull govardhanr992/java-web:${BUILD_NUMBER}'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@10.0.1.24 docker rm -f webserver || true'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@10.0.1.24 docker run -d -p 8080:8080 --name webserver govardhanr992/java-web:${BUILD_NUMBER}'
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

