pipeline {
  agent any

  stages {
    stage (" Pull Repository Retail Store Sample ") {
      steps {
        script {
          sshagent(credentials: [SECRET]) {
            sh """ssh -o StrictHostKeyChecking=no ${MASTER_SERVER} << EOF
              cd ${MASTER_DIR_GIT}
              git checkout staging
              git pull testing staging
              exit
            EOF"""
          }
        }
      }
    }

    stage (" Code Analysis ") {
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
              -Dsonar.host.url=${SONAR_URL} \
              -Dsonar.login=${SONAR_TOKEN}
            """
          }
        }
      }
    }

    stage (" Pull and Push Docker Images ") {
      steps {
        sshagent(credentials: [SECRET]) {
          sh """ssh -o StrictHostKeyChecking=no ${MASTER_SERVER} << EOF
            cd ${MASTER_DIR}
            docker pull public.ecr.aws/aws-containers/retail-store-sample-ui:0.8.2
            docker tag public.ecr.aws/aws-containers/retail-store-sample-ui:0.8.2 crocoxolen/retail-store-sample-ui: latest
            docker push crocoxolen/retail-store-sample-ui: latest
            docker system prune -a -f
            exit
          EOF"""
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
