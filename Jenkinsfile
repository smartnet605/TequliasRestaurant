pipeline {
    agent any

    environment {
        GIT_REPO_URL = 'https://github.com/smartnet605/TequliasRestaurant.git' // Replace with your Git repo
        IIS_SERVER_IP = 'localhost:8086/home'                 // Replace with your IIS server IP
        DOCKER_IMAGE_PREFIX = 'myapp'                   // Prefix for Docker images
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GIT_REPO_URL}"]]
                ])
            }
        }

        stage('Build All Docker Images') {
            steps {
                script {
                    def dockerDirs = findDockerDirs()
                    for (dir in dockerDirs) {
                        def imageName = "${DOCKER_IMAGE_PREFIX}-${dir}"
                        sh """
                        docker build -t ${imageName} ${dir}
                        """
                    }
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                script {
                    def dockerDirs = findDockerDirs()
                    for (dir in dockerDirs) {
                        def imageName = "${DOCKER_IMAGE_PREFIX}-${dir}"
                        def containerName = "container-${dir}"
                        sh """
                        docker run -d --name ${containerName} -p 8080:80 ${imageName}
                        """
                    }
                }
            }
        }

        stage('Configure IIS') {
            steps {
                script {
                    sshagent(['IIS-Credentials-ID']) { // Replace with your Jenkins credentials ID
                        sh """
                        ssh admin@${IIS_SERVER_IP} "appcmd add site /name:'MyApp' /bindings:http/*:80: /physicalPath:'C:\\inetpub\\wwwroot\\myapp'"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "All Docker images built, containers started, and IIS configured successfully."
        }
        failure {
            echo "Pipeline execution failed."
        }
    }
}

def findDockerDirs() {
    def dirs = sh(script: "find . -type f -name 'Dockerfile' | xargs -n1 dirname", returnStdout: true).trim().split('\n')
    return dirs
}
