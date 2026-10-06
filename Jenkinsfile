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
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version'
                    sh 'mvn build-helper:parse-version versions:set\
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalversion} versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
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
                    def shellCmd = 'bash ./server-cmds.sh ${IMAGE_NAME}'
                    sshagent(['ec2-server-key']) {
                       sh "scp docker-compose.yaml ec2-user@34.228.161.225:/home/ec2-user"
                       sh "scp server-cmds.sh ec2-user@34.228.161.225:/home/ec2-user"
                       sh "ssh -o StrictHostKeyChecking=no ec2-user@34.228.161.225 ${shellCmd}" 
                    }
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsID: 'git-hub', passwordVariable:'PASS', usernameVariable: 'USER')])
                    sh 'git remote set-url origin https:$USER:$PASS@github.com/Titobid/jenkins-project-08.git'
                    sh 'git add .'
                    sh 'git commit -m "ci: version bump"'
                    sh 'git push origin Head:jenkins-jobs'
                }
            }
        }   
    }
} 
