#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://github.com/Titobid/Jenkins-shared-library.git',
    credentialsID: 'git-hub'
    ]
)

pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        IMAGE_NAME= 'titobid/jenkins-app:java-maven-1.0'
    }
    stages {
        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 
        stage("deploy") {
            steps {
                script {
                    def dockerCmd = 'docker run -p 8080:8080 -d titobid/jenkins-app:1.0'
                    sshagent(['ec2-server-key']) {
                       sh "ssh -o StrictHostKeyChecking=no ec2-user@34.228.161.225 ${dockerCmd}"      
                    }
                }
            }
        }               
    }
} 
