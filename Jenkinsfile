pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'c1269293-12a7-4ae4-a1fe-b048736d5658'
        SLACK_CHANNEL = '#pipeline-notifications'
        SLACK_TOKEN_CREDENTIAL_ID = 'slack-token'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout your source code from your GitHub repository
                    def scmVars = checkout scm: [
                        $class: 'GitSCM',
                        branches: [[name: '*/develop']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        userRemoteConfigs: [[url: 'https://github.com/imzainazm/microservices-chat-demo.git']]
                    ]
                    currentBuild.description = "Triggered by: ${scmVars.GIT_AUTHOR_NAME}"
                }
            }
        }

        stage('Determine Changes') {
            steps {
                script {
                    // Determine which services have changed
                    def changedFiles = sh(
                        script: 'git diff --name-only HEAD~1 HEAD',
                        returnStdout: true
                    ).trim().split('\n')
                    
                    env.CHANGED_SERVICES = []
                    if (changedFiles.any { it.startsWith('api-gateway/') }) {
                        env.CHANGED_SERVICES += 'api-gateway'
                        echo "Changed Services: ${env.CHANGED_SERVICES}"
                    }
                    if (changedFiles.any { it.startsWith('users-service/') }) {
                        env.CHANGED_SERVICES += 'users-service'
                    }
                    if (changedFiles.any { it.startsWith('chat-service/') }) {
                        env.CHANGED_SERVICES += 'chat-service'
                    }
                    if (changedFiles.any { it.startsWith('chat-app/') }) {
                        env.CHANGED_SERVICES += 'chat-app'
                    }
                }
            }
        }

        stage('Build Docker Images') {
            when {
                expression { env.CHANGED_SERVICES }
            }
            parallel {
                stage('Build API Gateway') {
                    when {
                        expression { env.CHANGED_SERVICES.contains('api-gateway') }
                    }
                    steps {
                        script {
                            sh 'docker build -t imzainazm/api-gateway:latest ./api-gateway'
                        }
                    }
                }
                stage('Build Users Service') {
                    when {
                        expression { env.CHANGED_SERVICES.contains('users-service') }
                    }
                    steps {
                        script {
                            sh 'docker build -t imzainazm/users-service:latest ./users-service'
                        }
                    }
                }
                stage('Build Chat Service') {
                    when {
                        expression { env.CHANGED_SERVICES.contains('chat-service') }
                    }
                    steps {
                        script {
                            sh 'docker build -t imzainazm/chat-service:latest ./chat-service'
                        }
                    }
                }
                stage('Build Chat App') {
                    when {
                        expression { env.CHANGED_SERVICES.contains('chat-app') }
                    }
                    steps {
                        script {
                            sh 'docker build -t imzainazm/chat-app:latest ./chat-app'
                        }
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            when {
                expression { env.CHANGED_SERVICES }
            }
            parallel {
                stage('Push API Gateway') {
                    when {
                        expression { env.CHANGED_SERVICES.contains('api-gateway') }
                    }
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                                docker.image('imzainazm/api-gateway:latest').push()
                            }
                        }
                    }
                }
                stage('Push Users Service') {
                    when {
                        expression { env.CHANGED_SERVICES.contains('users-service') }
                    }
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                                docker.image('imzainazm/users-service:latest').push()
                            }
                        }
                    }
                }
                stage('Push Chat Service') {
                    when {
                        expression { env.CHANGED_SERVICES.contains('chat-service') }
                    }
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                                docker.image('imzainazm/chat-service:latest').push()
                            }
                        }
                    }
                }
                stage('Push Chat App') {
                    when {
                        expression { env.CHANGED_SERVICES.contains('chat-app') }
                    }
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                                docker.image('imzainazm/chat-app:latest').push()
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                slackSend(
                    botUser: true,
                    channel: SLACK_CHANNEL,
                    color: '#00ff00',
                    message: "Pipeline Succeeded\nTriggered by: "${env.BUILD_USER}"\nUpdated Services: ${env.CHANGED_SERVICES.join(', ')}\nEnvironment: ${env.JOB_NAME}",
                    tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
                )
            }
        }
        failure {
            script {
                slackSend(
                    botUser: true,
                    channel: SLACK_CHANNEL,
                    color: '#ff0000',
                    message: "Pipeline Failed\nTriggered by: ${env.BUILD_USER}\nUpdated Services: ${env.CHANGED_SERVICES.join(', ')}\nEnvironment: ${env.JOB_NAME}",
                    tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
                )
            }
        }
    }
