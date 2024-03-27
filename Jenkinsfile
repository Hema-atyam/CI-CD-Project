pipeline {
    agent any
    
    tools {
        maven 'maven'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_ACCESS_KEY_ID = credentials("access_key")
        AWS_SECRET_ACCESS_KEY = credentials("secret_key")
        AWS_DEFAULT_REGION="us-east-1"
    }

    stages {
        stage('Code Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/Hema-atyam/CI-CD-Project.git' 
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true" 
            }
        }
        
        stage('Sonar analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=CI-CD-Project -Dsonar.projectName=CI-CD-Project \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('OWASP DC') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploying to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        stage('docker build') {
            steps {
                sh "docker build -t hemaatyam/ekart-app:${BUILD_NUMBER} ."
            }
        }
        
        
        stage('trivy scan') {
            steps {
                sh "trivy image hemaatyam/ekart-app:${BUILD_NUMBER} > trivy-report.txt"
            }
        }
        
        stage('pushing image to dockerhub') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh "docker push hemaatyam/ekart-app:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('deploy-to-k8s') {
            steps {
                sh "aws eks update-kubeconfig --name demo --region us-east-1"
                sh "kubectl apply -f  . "
            }
        }
        
    }
}
