pipeline {
    agent any
    environment {
        DOCKER_PATH = "/usr/local/bin"
        PATH = "${DOCKER_PATH}:${env.PATH}"
        DOCKER_IMAGE = "tdlreinis/python-greetings-app:latest"
        API_TESTS_IMAGE = "tdlreinis/api-tests:latest"
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh '''
                    if [ -d "python-greetings" ]; then
                        echo "Directory 'python-greetings' exists. Removing it..."
                        rm -rf python-greetings
                    fi
                    echo "Cloning the repository..."
                    git clone https://github.com/PReinis/python-greetings.git
                    '''
                }
            }
        }
        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                        echo "Logging in to Docker Hub..."
                        echo $DOCKER_PASSWORD | $DOCKER_PATH/docker login -u $DOCKER_USERNAME --password-stdin
                        '''
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    echo "Building Docker image for Python service..."
                    $DOCKER_PATH/docker build -t ${DOCKER_IMAGE} ./python-greetings
                    $DOCKER_PATH/docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }
        stage('Deploy to DEV') {
            steps {
                script {
                    sh '''
                    echo "Deploying to DEV environment..."
                    cd python-greetings
                    $DOCKER_PATH/docker pull ${DOCKER_IMAGE}
                    $DOCKER_PATH/docker-compose down
                    $DOCKER_PATH/docker-compose up -d greetings-app-dev
                    '''
                }
            }
        }
        stage('Test on DEV') {
            steps {
                script {
                    sh '''
                    echo "Testing DEV environment..."
                    $DOCKER_PATH/docker pull ${API_TESTS_IMAGE}
                    $DOCKER_PATH/docker run --network=host --rm ${API_TESTS_IMAGE} run greetings greetings_dev
                    '''
                }
            }
        }
        stage('Deploy to STG') {
            steps {
                script {
                    sh '''
                    echo "Deploying to STG environment..."
                    cd python-greetings
                    $DOCKER_PATH/docker pull ${DOCKER_IMAGE}
                    $DOCKER_PATH/docker-compose down
                    $DOCKER_PATH/docker-compose up -d greetings-app-stg
                    '''
                }
            }
        }
        stage('Test on STG') {
            steps {
                script {
                    sh '''
                    echo "Testing STG environment..."
                    $DOCKER_PATH/docker pull ${API_TESTS_IMAGE}
                    $DOCKER_PATH/docker run --network=host --rm ${API_TESTS_IMAGE} run greetings greetings_stg
                    '''
                }
            }
        }
        stage('Deploy to PROD') {
            steps {
                script {
                    sh '''
                    echo "Deploying to PROD environment..."
                    cd python-greetings
                    $DOCKER_PATH/docker pull ${DOCKER_IMAGE}
                    $DOCKER_PATH/docker-compose down
                    $DOCKER_PATH/docker-compose up -d greetings-app-prod
                    '''
                }
            }
        }
        stage('Test on PROD') {
            steps {
                script {
                    sh '''
                    echo "Testing PROD environment..."
                    $DOCKER_PATH/docker pull ${API_TESTS_IMAGE}
                    $DOCKER_PATH/docker run --network=host --rm ${API_TESTS_IMAGE} run greetings greetings_prod
                    '''
                }
            }
        }
    }
    post {
        always {
            script {
                sh '''
                echo "Cleaning up..."
                rm -rf python-greetings
                '''
            }
        }
        failure {
            echo "Pipeline failed. Please check the logs."
        }
        success {
            echo "Pipeline executed successfully!"
        }
    }
}
