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
	stage('Pull Notifier') {
            steps {
                script {
                    discordSend description: 'Pull notif', 
                                footer: 'Pull code from github berhasil', 
                                image: 'https://t4.ftcdn.net/jpg/00/88/85/97/360_F_88859742_3pcsH0QNgseXjj2Y8HeZSXJbHUb19bx2.jpg', 
                                link: env.BUILD_URL, 
                                result: currentBuild.currentResult, 
                                scmWebUrl: 'https://github.com/komarkun/wayshub-backend-komar.git', 
                                thumbnail: 'https://cdn.discordapp.com/attachments/1241391101848322081/1242064109566951485/KomarKUN.png?ex=664c79d8&is=664b2858&hm=0321fbe451c67094d13a0f471d48ae4be8b25feb14f3ae0a23000ec0e29e7d59&', 
                                title: env.JOB_NAME, 
                                webhookURL: DISCORD_WEBHOOK_URL
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
	stage('Build Notifier') {
            steps {
                script {
                    discordSend description: 'Build Notif', 
                                footer: 'Build Berhasill', 
                                image: 'https://t4.ftcdn.net/jpg/00/88/85/97/360_F_88859742_3pcsH0QNgseXjj2Y8HeZSXJbHUb19bx2.jpg', 
                                link: env.BUILD_URL, 
                                result: currentBuild.currentResult, 
                                scmWebUrl: 'https://github.com/komarkun/wayshub-backend-komar.git', 
                                thumbnail: 'https://cdn.discordapp.com/attachments/1241391101848322081/1242064109566951485/KomarKUN.png?ex=664c79d8&is=664b2858&hm=0321fbe451c67094d13a0f471d48ae4be8b25feb14f3ae0a23000ec0e29e7d59&', 
                                title: env.JOB_NAME, 
                                webhookURL: DISCORD_WEBHOOK_URL
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
	stage('Test Notifier') {
            steps {
                script {
                    discordSend description: 'Test Notif', 
                                footer: 'Runs test wget spider berhasil', 
                                image: 'https://t4.ftcdn.net/jpg/00/88/85/97/360_F_88859742_3pcsH0QNgseXjj2Y8HeZSXJbHUb19bx2.jpg', 
                                link: env.BUILD_URL, 
                                result: currentBuild.currentResult, 
                                scmWebUrl: 'https://github.com/komarkun/wayshub-backend-komar.git', 
                                thumbnail: 'https://cdn.discordapp.com/attachments/1241391101848322081/1242064109566951485/KomarKUN.png?ex=664c79d8&is=664b2858&hm=0321fbe451c67094d13a0f471d48ae4be8b25feb14f3ae0a23000ec0e29e7d59&', 
                                title: env.JOB_NAME, 
                                webhookURL: DISCORD_WEBHOOK_URL
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
	stage('Deploy Notifier') {
            steps {
                script {
                    discordSend description: 'Deploy Notif', 
                                footer: 'deploy di server berhasil', 
                                image: 'https://t4.ftcdn.net/jpg/00/88/85/97/360_F_88859742_3pcsH0QNgseXjj2Y8HeZSXJbHUb19bx2.jpg', 
                                link: env.BUILD_URL, 
                                result: currentBuild.currentResult, 
                                scmWebUrl: 'https://github.com/komarkun/wayshub-backend-komar.git', 
                                thumbnail: 'https://cdn.discordapp.com/attachments/1241391101848322081/1242064109566951485/KomarKUN.png?ex=664c79d8&is=664b2858&hm=0321fbe451c67094d13a0f471d48ae4be8b25feb14f3ae0a23000ec0e29e7d59&', 
                                title: env.JOB_NAME, 
                                webhookURL: DISCORD_WEBHOOK_URL
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

	stage('Push Notifier') {
            steps {
                script {
                    discordSend description: 'Push Notif', 
                                footer: 'Berhasil push ke docker hub', 
                                image: 'https://t4.ftcdn.net/jpg/00/88/85/97/360_F_88859742_3pcsH0QNgseXjj2Y8HeZSXJbHUb19bx2.jpg', 
                                link: env.BUILD_URL, 
                                result: currentBuild.currentResult, 
                                scmWebUrl: 'https://github.com/komarkun/wayshub-backend-komar.git', 
                                thumbnail: 'https://cdn.discordapp.com/attachments/1241391101848322081/1242064109566951485/KomarKUN.png?ex=664c79d8&is=664b2858&hm=0321fbe451c67094d13a0f471d48ae4be8b25feb14f3ae0a23000ec0e29e7d59&', 
                                title: env.JOB_NAME, 
                                webhookURL: DISCORD_WEBHOOK_URL
                }
            }
        }

    }
}

