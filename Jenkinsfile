#!groovy

pipeline {
	agent { node ("af-stage-all01") }
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
        echo "=====================Removing====================="
	sh "cd docker/stage ; docker-compose down" 
	sh "docker rmi avalancheforecast/main:pipe_web_stage_parser"
	sh "docker rmi avalancheforecast/main:pipe_web_stage_frontend"
	sh "docker rmi avalancheforecast/main:pipe_web_stage_backend"
	echo "Done!"
      } catch (err) {
                echo "Контейнера не было"
            }
    }
		stage("Run container from Dockerhub") {
			steps {
				echo "=====================Running====================="
				sh "cd docker/stage ; docker-compose up -d "
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
