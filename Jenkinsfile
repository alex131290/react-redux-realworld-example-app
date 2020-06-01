pipeline {
    agent {label 'ec2-base-slave'}
    environment {
        SERVICE_NAME = "frontend"
        AWS_REGION = "us-east-1"
        AWS_ACCOUNT = "561306761274"
    }
    options { timestamps () }
    stages {

        // stage('Set display name and slack notification') {
        //     steps {
        //         wrap([$class: 'BuildUser']) {
        //             script {
        //                 currentBuild.displayName = "#${currentBuild.number} (user: ${BUILD_USER})"
        //             }
        //             slackSend color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' by ${BUILD_USER} (${env.BUILD_URL})"
        //         }
        //     }
        // }

        stage('Start Docker Service') {
            steps {
                sh '''sudo service docker start'''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                echo "Building ${SERVICE_NAME}"
                
                sudo docker build --build-arg IS_PROD=true -t "${SERVICE_NAME}" .
                sudo docker tag ${SERVICE_NAME}:latest "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${SERVICE_NAME}:${BUILD_NUMBER}"
                '''
            }
        }

        stage ('Push Docker') {
            steps {
                sh '''
                sudo $(aws ecr get-login --no-include-email --region ${AWS_REGION})
                sudo docker push ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${SERVICE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage ('Tag and Push') {
            environment {
                GIT_TAG = "BUILD_${BUILD_NUMBER}"
            }
            steps {
                sh('''
                    git tag -a ${GIT_TAG} -m "[Jenkins CI] Release version ${GIT_TAG}"
                ''')
              sshagent(['JenkinsGit']) {
                    sh("""
                        #!/usr/bin/env bash
                        set +x
                        export GIT_SSH_COMMAND="ssh -oStrictHostKeyChecking=no"
                        git push origin \$GIT_TAG
                     """)
                }
            }
        }
    }
    // post {
    //     success {
    //         wrap([$class: 'BuildUser']) {
    //             slackSend color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' by ${BUILD_USER} (${env.BUILD_URL})"
    //         }
    //     }
    //     failure {
    //         wrap([$class: 'BuildUser']) {
    //             slackSend color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' by ${BUILD_USER} (${env.BUILD_URL})"
    //         }
    //     }
    // }
}
