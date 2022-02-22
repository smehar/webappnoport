/* import shared library */
@Library('smehar-shared-library')_

pipeline {
     environment {
       IMAGE_NAME = "webappnoport"
       IMAGE_TAG = "latest"
       STAGING = "s-mehar-stag"
       PRODUCTION = "s-mehar-prod"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t smehar/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker stop ${IMAGE_NAME}
                    docker rm -f ${IMAGE_NAME}
                    docker run --name $IMAGE_NAME -d -p 81:81 -e PORT=81 smehar/$IMAGE_NAME:$IMAGE_TAG
                    sleep 10
                 '''
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                    docker stop ${IMAGE_NAME}
                    docker rm -f ${IMAGE_NAME}
                    docker run --name $IMAGE_NAME -d -p 81:81 -e PORT=81 smehar/$IMAGE_NAME:$IMAGE_TAG
                    sleep 10
                    curl http://localhost | grep "Welcome"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                 docker stop $IMAGE_NAME
                 docker rm $IMAGE_NAME
               '''
             }
          }
     }
     stage('Push image in staging and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              heroku container:login
              heroku create $STAGING || echo "project already exist"
              heroku container:push -a $STAGING web
              heroku container:release -a $STAGING web
            '''
          }
        }
     }
     stage('Push image in production and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              heroku container:login
              heroku create $PRODUCTION || echo "project already exist"
              heroku container:push -a $PRODUCTION web
              heroku container:release -a $PRODUCTION web
            '''
          }
        }
     }
  }
  post {
       always {
       script {
         slackNotifier currentBuild.result
     }
    }  
    }
}
