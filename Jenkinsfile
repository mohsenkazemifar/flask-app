pipeline {
    agent any

    environment {
        REGISTRY_URL = "192.168.2.249:5000"
        REGISTRY_CREDENTIALS = credentials('docker-registry')
        IMAGE_NAME = "${REGISTRY_URL}/flask-app"
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
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                dir('flask-app') {
                    sh '''
                        . venv/bin/activate
                        coverage run -m pytest
                        coverage xml
                    '''
                }
            }
        }

        stage('Sonar Analysis') {
            steps {
                dir('flask-app') {
                    withSonarQubeEnv('sonar') {
                        sh '''
                            export PATH=$PATH:/opt/sonar-scanner/bin
                            sonar-scanner \
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

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('flask-app') {
                    script {
                        docker.build("${IMAGE_NAME}:${IMAGE_TAG}", '.')
                    }
                }
            }
        }

        stage('Push to Private Registry') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY_URL}", REGISTRY_CREDENTIALS) {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
    }
}
