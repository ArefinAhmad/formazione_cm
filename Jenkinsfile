pipeline {
  agent any
  environment {
    registry = "localhost:5000"
    imageName = "myapp"   // sostituisci con il nome della tua app Dockerfile
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
          // usa il Dockerfile che vuoi (qui un esempio generico)
          sh "docker build -t ${registry}/${imageName}:${BUILD_NUMBER} -f templates/Dockerfile.ubuntu ."
        }
      }
    }
    stage('Push Image') {
      steps {
        dir("${WORKDIR}") {
          sh "docker push ${registry}/${imageName}:${BUILD_NUMBER}"
          sh "docker tag ${registry}/${imageName}:${BUILD_NUMBER} ${registry}/${imageName}:latest || true"
          sh "docker push ${registry}/${imageName}:latest || true"
        }
      }
    }
    stage('Deploy with Ansible') {
      steps {
        dir("${WORKDIR}") {
          sh """
            # esegui playbook deploy; inventory e chiave sono nella repo montata
            ansible-playbook -i inventory.ini playbooks/deploy.yml -e image=${registry}/${imageName}:${BUILD_NUMBER}
          """
        }
      }
    }
  }
}
