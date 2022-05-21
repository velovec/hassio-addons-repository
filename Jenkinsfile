pipeline {
    agent {
        docker {
            image 'ubuntu:20.04'
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
                sh "curl https://getcas.codenotary.io -L | sh"
            }
        }

        stage("Build") {
            steps {
                script {
                    def files = findFiles()

                    for (int i = 0; i < files.length; i++) {
                        if (files[i].directory) {
                            dir("${files[i].name}") {
                                stage ("Build :: Build ${files[i].name} Add-on") {
                                    steps {
                                        sh "docker build -t ${env.DOCKER_REPOSITORY}/${files[i].name}:latest ."
                                    }
                                }

                                stage ("Promote :: Promote ${files[i].name} Add-on") {
                                    when {
                                        branch 'master'
                                    }

                                    steps {
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
    }

    post {
        always {
            sh 'docker rmi $(docker images -f dangling=true -q) || echo "No dangling images"'

            deleteDir()
        }
    }
}