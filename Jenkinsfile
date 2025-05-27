pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME = "mohsenkazemifar392/flask-app1"
        IMAGE_TAG = "latest"
    }
    stages {
        stage('Fetch code') {
            steps {
                git branch: 'main', url: 'https://github.com/mohsenkazemifar/flask-app.git'
            }
        }
        stage('Install dependencies') {
            steps {
                dir('flask-app') {
                    sh 'python3 -m venv venv'
                    sh '. venv/bin/activate && pip install -r requirements.txt'
                }
            }
        }
        stage('Test') {
            steps {
                dir('flask-app') {
                    sh '. venv/bin/activate && coverage run -m pytest'
                    sh '. venv/bin/activate && coverage xml'
                }
            }
        }
        stage('Sonar Analysis') {
            steps {
                dir('flask-app') {
                    withSonarQubeEnv('sonar') {
                        sh '''
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=flask-app \
                            -Dsonar.projectName=flask-app \
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=. \
                            -Dsonar.python.coverage.reportPaths=coverage.xml
                        '''
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}", 'flask-app')
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
    }
}
