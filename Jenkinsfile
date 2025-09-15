pipeline {
    agent { label 'agent1' }
    environment {
        registry = "host.docker.internal:5000"
        imageName = "myapp"
        WORKDIR = "."
        DOCKERFILE = "templates/Dockerfile.agent"
        TARGET_SSH_KEY = "files/id_rsa_genericuser"
        TARGET_HOST = "127.0.0.1"
        TARGET_PORT = "2222"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }
        stage('Build Image') {
            steps {
                sh """
                    docker build -t ${registry}/${imageName}:${BUILD_NUMBER} -f ${DOCKERFILE} .
                    docker tag ${registry}/${imageName}:${BUILD_NUMBER} ${registry}/${imageName}:latest
                """
            }
        }
        stage('Push Image') {
            steps {
                sh """
                    docker push ${registry}/${imageName}:${BUILD_NUMBER}
                    docker push ${registry}/${imageName}:latest
                """
            }
        }
        
        stage('Deploy with Ansible') {
            steps {
                sh """
                    ansible-playbook -i inventory.ini playbooks/container-deploy.yml \
                    -e image=${registry}/${imageName}:${BUILD_NUMBER} \
                    --private-key ${TARGET_SSH_KEY}
                """
            }
        }
        stage('Verify SSH') {
            steps {
                sh """
            timeout 5 bash -c 'cat < /dev/null > /dev/tcp/127.0.0.1/2222'
        """
            }
        }
    }
}
