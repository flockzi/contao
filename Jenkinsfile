pipeline {

    agent any

    triggers { 
        pollSCM('H/15 * * * *')
    }

    environment {
        dockerImage = ''
        registryName = "${JOB_NAME}"
        GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%h'", returnStdout: true)
    }

    stages {

        stage('cleanup') {
            steps {
            // Recursively delete all files and folders in the workspace
            // using the built-in pipeline command
                deleteDir()
            }
        }
        
        stage('Checkout') {
            steps {
                echo 'Pulling...' + env.BRANCH_NAME
                echo 'Pulling...' + scm.branches[0].name
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("ortinteractive/${registryName}")
                }
            }
        }

        //stage("phpunit") {
        //    steps {
        //        script {
        //            dockerImage.inside {
        //                sh './vendor/bin/phpunit --testdox tests'
        //            }
        //        }
        //    }
        //}

        stage('Push image') {
            steps {
                script {
                    docker.withRegistry('https://registry.ort.services', 'registry') {
                    dockerImage.push("${env.BUILD_NUMBER}")
                    dockerImage.push("latest")
                    dockerImage.push(GIT_COMMIT_HASH)
                    }
                }
            }
        }
    }

    post {
        failure {
            slackSend channel: '#test',
                      color: 'bad',
                      message: "The pipeline ${currentBuild.fullDisplayName} failed."
        }
        success {
            slackSend channel: '#test',
                      color: 'good',
                      message: "The pipeline ${currentBuild.fullDisplayName} completed successfully."
        }
    }
}