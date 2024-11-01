pipeline {
  agent any

  stages {
    stage (" Pull Repository Retail Store Sample ") {
      steps {
        script {
          sshagent(credentials: [SECRET]) {
            sh """ssh -o StrictHostKeyChecking=no ${MASTER_SERVER} << EOF
              cd ${MASTER_DIR_GIT}
              git checkout production
              git pull testing production
              exit
            EOF"""
          }
        }
      }
    }

    stage ( "Code Analysis" ) {
      environment {
        scannerHome = tool name: 'sonarqube'
      }

      steps {
        script {
          withSonarQubeEnv('sonarqube') {
            sh """
              ${scannerHome}/bin/sonar-scanner \
              -Dsonar.projectKey=Retail-Store-Compose \
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
            docker pull public.ecr.aws/aws-containers/retail-store-sample-catalog:0.8.2
            docker tag public.ecr.aws/aws-containers/retail-store-sample-catalog:0.8.2 crocoxolen/retail-store-sample-catalog:latest
            docker push crocoxolen/retail-store-sample-ui:latest
            docker system prune -a -f
            exit
          EOF"""
        }
      }
    }
  }
}
