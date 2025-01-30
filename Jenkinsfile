/* Requires the Docker Pipeline plugin */
pipeline {
    agent {
        docker {
            image 'python:3.11.2-bullseye' // Debian-based image compatible with Raspberry Pi
        }
    }

    environment {
        REPO_URL = 'https://github.com/kottek-pmc/raspapi'
        APP_DIR = 'raspapi'
        DOCKER_IMAGE = 'raspapi:latest'
        IMAGE_TAR = 'raspapi.tar'
        REMOTE_HOST = '10.0.0.238'  // Change to Raspberry Pi IP
        REMOTE_USER = 'pmc'  // Change if needed
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    sh 'rm -rf ${APP_DIR}'
                    sh "git clone ${REPO_URL} ${APP_DIR}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        cd ${APP_DIR}
                        docker build -t ${DOCKER_IMAGE} .
                    """
                }
            }
        }

        stage('Save & Transfer Image') {
            steps {
                script {
                    sh """
                        docker save -o ${IMAGE_TAR} ${DOCKER_IMAGE}
                        scp ${IMAGE_TAR} ${REMOTE_USER}@${REMOTE_HOST}:/home/${REMOTE_USER}/
                    """
                }
            }
        }

        stage('Deploy on Raspberry Pi') {
            steps {
                script {
                    sh """
                        ssh ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
                            docker load -i /home/${REMOTE_USER}/${IMAGE_TAR}
                            docker stop raspapi || true
                            docker rm raspapi || true
                            docker run -d --name raspapi -p 8000:8000 raspapi:latest
                            rm /home/${REMOTE_USER}/${IMAGE_TAR}  # Clean up transferred image
                        EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}