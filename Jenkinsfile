pipeline {
  agent { label 'agent1' }  // usa il nodo agent1
  environment {
    registry = "host.docker.internal:5000"  // su Mac Docker Desktop
    imageName = "myapp"
    WORKDIR = "formazione_cm"
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
            docker build -t ${registry}/${imageName}:${BUILD_NUMBER} -f templates/Dockerfile.ubuntu .
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
    }
    stage('Deploy with Ansible') {
      steps {
        dir("${WORKDIR}") {
          sh """
            ansible-playbook -i inventory.ini playbooks/deploy.yml -e image=${registry}/${imageName}:${BUILD_NUMBER} --private-key files/id_rsa_genericuser
          """
        }
      }
    }
  }
}

