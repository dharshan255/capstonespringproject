pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'dharshan18/pipeline'
        GIT_REPO = 'https://github.com/dharshan255/capstonespringproject'
    }

    tools {
        maven 'Maven' // Make sure this is configured in Jenkins Global Tool Configuration
    }

    stages {
        stage('Clone') {
            steps {
                echo 'Cloning repository...'
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build') {
            steps {
                echo 'Running Maven build...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image...'
                withCredentials([
                    usernamePassword(
                        credentialsId: 'b3448268-dd9b-4bc2-a6fe-f83a8dad1a8c',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh """
                        echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    withCredentials([
                        sshUserPrivateKey(
                            credentialsId: 'ec2-ssh-key',
                            keyFileVariable: 'KEYFILE'
                        )
                    ]) {
                        sh """
                            mkdir -p ~/.ssh
                            ssh-keyscan -H 54.85.104.120 >> ~/.ssh/known_hosts
                            ssh -i \$KEYFILE ec2-user@54.85.104.120 '
                                sudo docker pull ${DOCKER_IMAGE}:${BUILD_NUMBER} &&
                                sudo docker stop spring-petclinic || true &&
                                sudo docker rm spring-petclinic || true &&
                                sudo docker run -d --name spring-petclinic -p 8081:8080 ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            '
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
