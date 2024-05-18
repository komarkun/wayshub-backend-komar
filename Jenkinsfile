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
	stage('Discord Notifier') {
            steps {
                script {
                    discordSend description: 'Jobs CI/CD aplikasi backend', 
                                footer: 'Healing atuh', 
                                image: 'https://t4.ftcdn.net/jpg/00/88/85/97/360_F_88859742_3pcsH0QNgseXjj2Y8HeZSXJbHUb19bx2.jpg', 
                                link: env.BUILD_URL, 
                                result: currentBuild.currentResult, 
                                scmWebUrl: 'https://github.com/komarkun/wayshub-backend-komar.git', 
                                thumbnail: 'https://t4.ftcdn.net/jpg/00/88/85/97/360_F_88859742_3pcsH0QNgseXjj2Y8HeZSXJbHUb19bx2.jpg', 
                                title: env.JOB_NAME, 
                                webhookURL: DISCORD_WEBHOOK_URL
                }
            }
        }

    }
}

