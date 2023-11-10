pipeline {
    agent any
    
    tools{
        maven 'maven-3'   
        jdk 'jdk17'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
               git 'https://github.com/jaiswaladi246/secretsanta-generator.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Tests ') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=santa \
                     -Dsonar.projectKey=santa -Dsonar.java.binaries=. '''
                }            
        }
        }
        stage('Owasp scan') {
            steps {
                dependencyCheck additionalArguments: ' --scan .', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build Application ') {
            steps {
                sh "mvn package"
            }
        }
        
    }
}
                

