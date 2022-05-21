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
        stage("Install CAS") {
            steps {
                sh "bash <(curl https://getcas.codenotary.io -L)"
            }
        }

        def files = findFiles()
        files.each(f -> {
            if (f.directory) {
                dir("${f.name}") {
                    stage ("Build :: Build ${f.name} Add-on") {
                        steps {
                            sh "docker build -t ${env.DOCKER_REPOSITORY}/${f.name}:latest ."
                        }
                    }

                    stage ("Promote :: Promote ${f.name} Add-on") {
                        when {
                            branch 'master'
                        }

                        steps {
                            sh "cas notarize docker://${env.DOCKER_REPOSITORY}/${f.name}:latest"
                            sh "docker push ${env.DOCKER_REPOSITORY}/${f.name}:latest"
                        }
                    }
                }
            }
        })

    }

    post {
        always {
            sh 'docker rmi $(docker images -f dangling=true -q) || echo "No dangling images"'

            deleteDir()
        }
    }
}