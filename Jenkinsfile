def secret = 'vm'
def branch = 'master'
def serverCredentialsId = 'server'
def directoryCredentialsId = 'directory'
def dockerLoginCredentialsId = 'docker-login'
def nameBuildCredentialsId = 'namebuild'

pipeline {
    agent any
    environment {
	SERVER = credentials("${serverCredentialsId}")
	DIRECTORY = credentials("${directoryCredentialsId}")
	DOCKERLOGIN = credentials("${dockerLoginCredentialsId}")
	NAMEBUILD = credentials("${nameBuildCredentialsId}")
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
    }
}

