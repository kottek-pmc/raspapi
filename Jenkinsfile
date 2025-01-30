/* Requires the Docker Pipeline plugin */
pipeline {
    agent any

    environment {
        REPO_URL    = 'https://github.com/kottek-pmc/raspapi'
        APP_DIR     = 'raspapi'
        DOCKER_IMAGE= 'raspapi:latest'
        IMAGE_TAR   = 'raspapi.tar'
        REMOTE_HOST = '10.0.0.238'
        REMOTE_USER = 'pmc'
    }

    stages {
        stage('Checkout Code') {
            steps {
                sh 'rm -rf ${APP_DIR}'
                sh "git clone ${REPO_URL} ${APP_DIR}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Optionally create a Dockerfile if the repo doesn't have one
                    sh """
                        cd ${APP_DIR}
                        # Only create a Dockerfile if one doesn't exist already
                        if [ ! -f Dockerfile ]; then
                            cat <<EOF > Dockerfile
FROM python:3.11.2-bullseye
WORKDIR /app
COPY requirements.txt /app
RUN pip install --no-cache-dir -r requirements.txt
COPY . /app
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
EOF
                        fi
                    """

                    // Now build
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
                            docker run -d --name raspapi -p 8000:8000 ${DOCKER_IMAGE}
                            rm /home/${REMOTE_USER}/${IMAGE_TAR}
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