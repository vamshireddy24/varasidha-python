pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                sh 'echo passed'
                 git branch: 'main', url: "https://github.com/vamshireddy24/varasiddhi-python.git"
            }
        }
        stage('Install Dependencies') {
            steps {
                // Install Python dependencies
                sh '''
                . ${VENV_DIR}/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                // Run tests
                sh '''
                . ${VENV_DIR}/bin/activate
                pytest --maxfail=1 --disable-warnings -q
                '''
            }
        }
         stage('Sonar-Test') {
             environment {
                 SONAR_URL = "http://localhost:9000"
             }
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                        // Run SonarQube Scanner
                        sh '''
                        . ${VENV_DIR}/bin/activate
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${SONAR_URL} \
                          -Dsonar.login=${SONAR_AUTH_TOKEN}
                        '''
                    }
                }
            }
        }
        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "kubevamshi/varasiddha-py:${BUILD_NUMBER}"
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
