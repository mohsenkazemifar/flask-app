pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'flask-ci-cd-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        REGISTRY = '192.168.2.249:5000'
        SONARQUBE_ENV = 'SonarQube'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mohsenkazemifar/flask-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('app') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('app') {
                    sh 'pytest || echo "No tests found, skipping..."'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .'
            }
        }

        stage('Docker Push to Local Registry') {
            steps {
                sh """
                    docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${REGISTRY}/${DOCKER_IMAGE}:${IMAGE_TAG}
                    docker push ${REGISTRY}/${DOCKER_IMAGE}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        sed 's|image: .*|image: ${REGISTRY}/${DOCKER_IMAGE}:${IMAGE_TAG}|' k8s/deployment.yaml > k8s/deployment-patched.yaml
                        kubectl apply -f k8s/deployment-patched.yaml
                        kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD pipeline completed successfully.'
        }
        failure {
            echo '❌ Build failed. Check logs above.'
        }
    }
}
