def secret = 'vm'
def branch = 'master'
def serverCredentialsId = 'server'
def directoryCredentialsId = 'directory'
def dockerLoginCredentialsId = 'docker-login'
def nameBuildCredentialsId = 'namebuild'
def discordurlCredentialsId = 'discord-webhook-url'
def appurlCredentialsId = 'app-url'

pipeline {
    agent any
    environment {
	SERVER = credentials("${serverCredentialsId}")
	DIRECTORY = credentials("${directoryCredentialsId}")
	DOCKERLOGIN = credentials("${dockerLoginCredentialsId}")
	NAMEBUILD = credentials("${nameBuildCredentialsId}")
	DISCORD_WEBHOOK_URL = credentials("${discordurlCredentialsId}")
	APPURL = credentials("${appurlCredentialsId}")
    }
    stages {
        stage ('pull new code') {
            steps {
                sshagent([secret]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SERVER} << EOF
                            cd ${DIRECTORY}
                            git pull origin ${branch}
                            echo "Selesai Pulling!"
                            exit
                        EOF
                    """
                }
            }
        }

        stage ('build the code') {
            steps {
                sshagent([secret]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SERVER} << EOF
                            cd ${DIRECTORY}
                            docker build -t ${NAMEBUILD} .
                            echo "Selesai Building!"
                            exit
                        EOF
                    """
                }
            }
        }

	stage('Test the app') {
            steps {
                sshagent([secret]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SERVER} << EOF
                            wget --spider --timeout=30 --tries=1 ${APPURL}
                            echo "Selesai Testing!"
                            exit
                        EOF
                    """
                }
            }
        }

        stage ('deploy') {
            steps {
                sshagent([secret]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SERVER} << EOF
                            cd ${DIRECTORY}
                            cd ../
                            docker compose -f docker-compose-backend.yaml down
                            docker compose -f docker-compose-backend.yaml up -d
                            echo "Selesai Men-Deploy!"
                            exit
                        EOF
                    """
                }

            }
        }
	stage ('Push to Docker Hub') {
            steps {
                sshagent([secret]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SERVER} << EOF
                            cd ${DIRECTORY}
			    ${DOCKERLOGIN}
			    docker push ${NAMEBUILD}
                            echo "Selesai Push Images To Docker Registries"
                            exit
                        EOF
                    """
                }

            }
        }

	stage('Send notification to Discord') {
            steps {
                script {
                    def message = "Deployment of ${NAMEBUILD} is complete and tested successfully!"
                    sh """
                        curl -H "Content-Type: application/json" \
                            -X POST \
                            -d '{"content": "${message}"}' \
                            ${DISCORD_WEBHOOK_URL}
                    """
                }
            }
        }
	stage('Discord Notifier from Extension plugins'){
	   steps {
	      discordSend(
 description: 'ini dari desckription', footer: 'ini dari footer', image: 'https://t3.ftcdn.net/jpg/02/93/59/72/360_F_293597295_lhk0X8DKsYarhMYguuWPP15qOtKOz0Qa.jpg', link: 'env.BUILD_URL', result: 'SUCCESS', scmWebUrl: 'https://github.com/komarkun/wayshub-backend-komar.git', thumbnail: 'https://t3.ftcdn.net/jpg/02/93/59/72/360_F_293597295_lhk0X8DKsYarhMYguuWPP15qOtKOz0Qa.jpg', title: 'env.JOB_NAME', webhookURL: 'https://discordapp.com/api/webhooks/1241391254621782171/MuahAQIXQDCBoR5gXvhK5po-WAxz9QtGcWOGz8-pwdxFOMwwzYPS_dHQyYu_oKktqggy'
		)
		}
	}
    }
}

