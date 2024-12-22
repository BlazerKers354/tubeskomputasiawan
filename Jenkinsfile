pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "blazerkers/tubescc:${BUILD_NUMBER}"
        // DOCKER_USERNAME = credentials("docker-tubes")
        // DOCKER_PASSWORD = credentials("docker-tubes")
        DISCORD_WEBHOOK = credentials("webhook-discord")
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clone the GitHub repository explicitly
                    git 'https://github.com/BlazerKers354/tubeskomputasiawan.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-tubes', 
                 usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat """
                    echo %DOCKER_PASSWORD% > docker-password.txt
                    docker login -u %DOCKER_USERNAME% --password-stdin < docker-password.txt
                    docker push ${DOCKER_IMAGE}
                    del docker-password.txt
                    """
                }
            }
        }

        stage('Notify Discord') {
            steps {
                script {
                    def message = [
                        "content": "Pipeline berhasil",
                        "embeds": [
                            [
                                "title": "docker build dan push",
                                "description": "Image `${DOCKER_IMAGE}` berhasil di push",
                                "color": 3066993
                            ]
                        ]
                    ]
                    httpRequest(
                        httpMode: 'POST',
                        acceptType: 'APPLICATION_JSON',
                        contentType: 'APPLICATION_JSON',
                        requestBody: groovy.json.JsonOutput.toJson(message),
                        url: DISCORD_WEBHOOK
                    )
                }
            }
        }
    }

    post {
        failure {
            script {
                def message = [
                    "content": "Pipeline gagal",
                    "embeds": [
                        [
                            "title": "Pipeline gagal",
                            "description": "Terdapat kesalahan",
                            "color": 15158332
                        ]
                    ]
                ]
                httpRequest(
                    httpMode: 'POST',
                    acceptType: 'APPLICATION_JSON',
                    contentType: 'APPLICATION_JSON',
                    requestBody: groovy.json.JsonOutput.toJson(message),
                    url: DISCORD_WEBHOOK
                )
            }
        }
    }
}
