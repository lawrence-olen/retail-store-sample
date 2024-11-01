pipeline {
  agent any

  stages {
    stage (" Pull Repository Retail Store Sample ") {
      steps {
        script {
          sshagent(credentials: [SECRET]) {
            sh """ssh -o StrictHostKeyChecking=no ${MASTER_SERVER} << EOF
              cd ${MASTER_DIR}
              git checkout main
              git pull origin main
              exit
            EOF"""
          }
        }
      }
    }

    stage (" Install Trivy ") {
      steps {
        script {
          sshagent(credentials: [SECRET]) {
            sh """
            wget https://github.com/aquasecurity/trivy/releases/download/v0.40.0/trivy_0.40.0_Linux-64bit.deb
            sudo dpkg -i trivy_0.40.0_Linux-64bit.deb
            echo "Trivy installed successfully"
            """
          }
        }
      }
    }

    stage (" Scan Images with Trivy") {
      steps {
        script {
          sshagent(credentials: [SECRET]) {
            sh"""
              # Scan Docker image for vulnerabilities
              trivy image --severity CRITICAL, HIGH crocoxolen/retail-store-sample-ui:latest
              echo "Trivy Scan Success"
            """
          }
        }
      }
    }

    stage (" Testing with wget spider ") {
      steps {
        script {
          sshagent(credentials: [SECRET]) {
            sh"""
            if wget --spider --server-response http://35.197.153.227:30009/ 2>&1 | grep -q "200 OK"; then
              echo "Aplikasi Berjalan"
            else
              echo "Aplikasi tidak berjalan"
              exit 1
            fi
            """
          }
        }
      }
    }

    stage (" Deploy Apps using Helmfile ") {
      steps {
        sshagent(credentials: [SECRET]) {
          sh """ssh -o StrictHostKeyChecking=no ${MASTER_SERVER} << EOF
            cd ${HELMFILE_DIR}
            helmfile apply
            echo "Deploy Apps Success"
            exit
          EOF"""
        }
      }
    }
  }
}