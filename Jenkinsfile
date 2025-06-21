pipeline {
  agent {
    label('slave')
  }

  environment {
    IMAGE_NAME = "meghanahs/case1:latest"
    MANIFEST_PATH = "manifest_file/k8s"
    DOCKER_HUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIALS')
  }  
  tools {
    git 'Default'
}
    stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/meghanahs1310/Case1_Repo.git'
      }
    }

    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        sh 'mvn clean install'
      }
    }
stage('Debug Credential') {
            steps {
                script {
                    echo "Resolved username: ${DOCKER_HUB_CREDENTIALS_USR}"
                    echo "Password (will be masked): ${DOCKER_HUB_CREDENTIALS_PSW}"
                }
            }
        }
    stage('Push to Docker Hub') {
      steps {
        script {
              echo "Cred ID: ${DOCKER_HUB_CREDENTIALS}"
          docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
            docker.image("${IMAGE_NAME}").push()
          }
        }
      }
    }

    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://65.2.3.174:9000/"
      }
      steps {
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
          sh """
            mvn sonar:sonar \
              -Dsonar.login=$SONAR_AUTH_TOKEN \
              -Dsonar.host.url=$SONAR_URL
          """
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        script {
          sh "kubectl apply -f ${MANIFEST_PATH}/dev/deployment.yaml --namespace=dev"
        }
      }
    }

    stage('Deploy to Test') {
      steps {
        script {
          sh """
            kubectl apply -f ${MANIFEST_PATH}/test/deployment.yaml --namespace=test
            kubectl rollout status deployment/case1 --namespace=test
          """
        }
      }
    }

    stage('Approval to Deploy to Prod') {
      steps {
        script {
          input(
            message: "Approve deployment to Prod?",
            parameters: [
              booleanParam(name: 'Proceed', defaultValue: false, description: 'Approve the deployment to Prod')
            ]
          )
        }
      }
    }

    stage('Deploy to Prod') {
      when {
        expression { return params.Proceed == true }
      }
      steps {
        script {
          sh """
            kubectl apply -f ${MANIFEST_PATH}/prod/deployment.yaml --namespace=prod
            kubectl rollout status deployment/case1 --namespace=prod
          """
        }
      }
    }
  }

  post {
    success {
      echo 'Deployment successful!'
      mail to: 'meghana.hs1400@gmail.com',
           subject: "Jenkins Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: "Good news! Jenkins job '${env.JOB_NAME}' (build #${env.BUILD_NUMBER}) completed successfully.\n\nCheck details: ${env.BUILD_URL}"
    }

    failure {
      echo 'Deployment failed!'
      mail to: 'meghana.hs1400@gmail.com',
           subject: "Jenkins Pipeline Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: "Oops! Jenkins job '${env.JOB_NAME}' (build #${env.BUILD_NUMBER}) failed.\n\nCheck details: ${env.BUILD_URL}"
    }
  }
}
