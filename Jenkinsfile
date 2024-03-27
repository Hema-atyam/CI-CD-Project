pipeline {
    agent any
    
    tools {
        maven 'maven'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
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
        
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        
        
    }
}
