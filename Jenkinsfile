pipeline {
    agent any
    tools {
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('git clone') {
            steps {
                git 'https://github.com/jetty251/ekart-project.git'
            }
        }
         stage('mvncompile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('mvntest') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('sonarqube') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=jettyekart -Dsonar.projectName=jettyekart -Dsonar.java.binaries=. "
                        
                    }
                }
            }
        }
       stage('mvnbuild') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('s3') {
            steps {
                 sh 'aws s3 cp /var/lib/jenkins/workspace/jettyekart/ s3://jetty-hotstar/jettyekart/ --recursive'
            }
        }
        stage('dockerbuild/tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                     sh "docker build -t jetty556677/jettyekart:v1 ."
                        
                    }
                }
            }
        }
        stage('dockerpush-dockerhub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                     sh "docker push jetty556677/jettyekart:v1" 
                    }
                }
         
            }
        }
        stage('docker-deploy') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                     sh "docker run -d -p 8070:8070 jetty556677/jettyekart:v1" 
                    } 
               }
            }
        }
        stage('kubernets') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'kube-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.95.162:6443') {
                        sh 'kubectl apply -f deploymentservice.yml -n webapps'
                        sh 'kubectl get svc -n webapps'
                    }
                }
            }
        }
    }
}
