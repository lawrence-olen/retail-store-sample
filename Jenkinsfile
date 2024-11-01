pipeline {
  agent any

  stages {
    stage (" Pull Repository Retail Store Sample ") {
      steps {
        script {
          sshagent(credentials: [SECRET]) {
            sh """ssh -o StrictHostKeyChecking=no ${MASTER_SERVER} << EOF
              cd ${MASTER_DIR}
              git checkout staging
              git pull testing staging
              exit
            EOF"""
          }
        }
      }
    }

    stage (" Code Analysisi ") {
      environment {
        scannerHome = tool name: 'sonarqube'
      }

      steps {
        script {
          withSonarQubeEnv('sonarqube') {
            sh """
              ${scannerHome}/bin/sonar-scanner \
              -Dsonar.projectKey=Retail-Store-Helmfile \
              -Dsonar.sources=. \
              -Dsonar.exclusions=**/*.java \
              -Dsonar.host.url=https://sonarqube-axiata.olen.studentdumbways.my.id \
              -Dsonar.login=squ_a37cc99b5d7c9a228290036b53ce06b797772475
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
