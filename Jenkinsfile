pipeline {
    agent { label 'agent1' } // nodo Jenkins agent già configurato
    environment {
        registry = "host.docker.internal:5000" // Docker registry locale
        imageName = "myapp"                    // nome dell'immagine
        WORKDIR = "."              // cartella del progetto
        DOCKERFILE = "templates/Dockerfile.agent"
        TARGET_SSH_KEY = "id_rsa_genericuser" // chiave SSH per Ansible
        TARGET_HOST = "127.0.0.1"
        TARGET_PORT = "2222"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                dir("${WORKDIR}") {
                    sh 'ls -la'
                }
            }
        }
        stage('Build Image') {
            steps {
                dir("${WORKDIR}") {
                    sh """
                        docker build -t ${registry}/${imageName}:${BUILD_NUMBER} -f ${DOCKERFILE} .
                        docker tag ${registry}/${imageName}:${BUILD_NUMBER} ${registry}/${imageName}:latest
                    """
                }
            }
        }
        stage('Push Image') {
            steps {
                dir("${WORKDIR}") {
                    sh """
                        docker push ${registry}/${imageName}:${BUILD_NUMBER}
                        docker push ${registry}/${imageName}:latest
                    """
                }
            }
            stage('Verify SSH') {
            steps {
                sh "nc -zv ${TARGET_HOST} ${TARGET_PORT}"
            }
        }
        stage('Deploy with Ansible') {
            steps {
                dir("${WORKDIR}") {
                    sh """
                        ansible-playbook -i inventory.ini playbooks/container-deploy.yml \
                        -e image=${registry}/${imageName}:${BUILD_NUMBER} \
                        --private-key files/id_rsa_genericuser
                    """
                }
            }
        }
    }
}
