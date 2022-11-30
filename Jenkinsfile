#!groovy

pipeline {
	agent { node ("avalanche") }
	options {
		buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
		timestamps()
	}
	stages {
		stage("Login Dockerhub") {
			steps {
				withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID'), string(credentialsId: 'avalancheforecast', variable: 'LOGIN'), string(credentialsId: 'avalanche_password', variable: 'PASS')]) {
                                sh  ("""
                                curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : *Branch*: ${env.GIT_BRANCH} *`Началось развертывание!`*'
                                """)
				echo "=====================Authentification====================="
				sh "docker login -u ${LOGIN} -p ${PASS}"
				}
			}
		}
    script{
      try{
        stage("Remove previous container version!") {
			steps {
				echo "=====================Removing====================="
        
				sh "docker stop web_stage_avalanche"
				sh "docker rm web_stage_avalanche"
				sh "docker rmi avalancheforecast/main:web_stage_avalanche"
				echo "Done!"
                                }
			}
      } catch (err) {
                echo "Контейнера не было"
            }
    }
		stage("Run container from Dockerhub") {
			steps {
				echo "=====================Running====================="
				sh "cd docker"
				sh "docker-compose up -d"
			}
		}
	}
	  post {
          success { 
          withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
          sh  ("""
              curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : `Официально задеплоен!)`'
          """)
          }
          }
          aborted {
          withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
          sh  ("""
              curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : `Отменен`'
          """)
          }
          }
          failure {
          withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
          sh  ("""
              curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC  *Branch*: ${env.GIT_BRANCH} *Build* : `Неудача :( Увы, что-то пошло не так`'
          """)
          }
          }
}
}
