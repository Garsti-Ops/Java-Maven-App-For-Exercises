#!/usr/bin/env groovy

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'garstiops/garstiges-secret-repo:java-maven-2.0'
    }
    stages {
        stage('build app') {
            steps {
                echo 'building application jar...'
                sh 'mvn package'

            }
        }
        stage('build image') {
            steps {
                echo 'building docker image...'
                sh 'docker build -t $IMAGE_NAME'
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
                sh 'docker push $IMAGE_NAME'
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image to EC2...'

                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    def ec2Instance = "ec2-user@35.180.251.121"

                    sshagent(['ec2-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        }
    }
}
