pipeline {
    agent any

    environment {
        REGISTRY_URL = "192.168.2.249:5000"
        IMAGE_NAME = "flask-app"
        IMAGE_TAG = "latest"
        FULL_IMAGE_NAME = "${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}"
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
                        docker.build("${FULL_IMAGE_NAME}", '.')
                    }
                }
            }
        }

        stage('Push to Private Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-registry', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        sh """
                            docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD ${REGISTRY_URL}
                            docker push ${FULL_IMAGE_NAME}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('flask-app') {
                    withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG_FILE')]) {
                        script {
                            sh '''
                                export KUBECONFIG=$KUBECONFIG_FILE
                                kubectl apply -f k8s/
                            '''
                        }
                    }
                }
            }
        }
    }
}
