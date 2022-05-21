pipeline {
    agent {
        docker {
            image 'docker.velovec.pro/ci-images/cas:latest'
            label 'docker'
            args '-u 1001:998 -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock -v ${HOME}/.docker:${WORKSPACE}/.docker'
        }
    }

    environment {
        DOCKER_REPOSITORY = 'docker.velovec.pro/hassio-addons'
        DOCKER_CONFIG = "${env.WORKSPACE}/.docker"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '15', daysToKeepStr: '30'))
    }

    stages {
        stage("Prerequisites") {
            steps {
                sh "cas login"
            }
        }

        stage("Build") {
            when {
                branch 'master'
            }

            steps {
                script {
                    def files = findFiles()

                    for (int i = 0; i < files.length; i++) {
                        if (files[i].directory && !files[i].name.startsWith(".")) {
                            dir("${files[i].name}") {
                                stage ("Build :: Build ${files[i].name} Add-on") {
                                    sh "docker build -t ${env.DOCKER_REPOSITORY}/${files[i].name}:latest ."
                                    sh "cas notarize docker://${env.DOCKER_REPOSITORY}/${files[i].name}:latest"
                                    sh "docker push ${env.DOCKER_REPOSITORY}/${files[i].name}:latest"
                                }
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker rmi $(docker images -f dangling=true -q) || echo "No dangling images"'

            deleteDir()
        }
    }
}