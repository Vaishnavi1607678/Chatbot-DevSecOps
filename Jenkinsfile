pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node19'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/NotHarshhaa/DevOps-Projects.git', branch: 'main'
                dir('DevOps-Project-28/Chatbot-UI') {
                    echo "Checked out Chatbot UI source"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('DevOps-Project-28/Chatbot-UI') {
                    sh 'npm install'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    dir('DevOps-Project-28/Chatbot-UI') {
                        sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=ChatbotUI \
                        -Dsonar.projectName=ChatbotUI
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dir('DevOps-Project-28/Chatbot-UI') {
                    dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }

        stage('Docker Build & Run') {
            steps {
                dir('DevOps-Project-28/Chatbot-UI') {
                    script {
                        sh 'docker build -t chatbot-ui .'
                        sh 'docker stop chatbot-ui || true'
                        sh 'docker rm chatbot-ui || true'
                        sh 'docker run -d -p 3000:3000 --name chatbot-ui chatbot-ui'
                    }
                }
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh 'trivy image chatbot-ui > trivy_report.txt || true'
            }
        }
    }
}
