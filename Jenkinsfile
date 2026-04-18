pipeline {
    agent any

    environment {
        SONAR_URL = "https://32.192.187.102"
        DOCKER_REGISTRY = "32.192.187.102:8082"
        IMAGE_NAME = "demo-app"
    }

    tools {
        maven 'maven3'
        jdk 'jdk21'
    }

    stages {

        stage('Checkout') {
            steps {
                git credentialsId: 'github-ssh', url: 'git@github.com:<YOUR-USERNAME>/<YOUR-REPO>.git'
            }
        }

        stage('Build') {
            steps {
                dir('app') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('app') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'TOKEN')]) {
                        sh """
                        mvn sonar:sonar \
                        -Dsonar.host.url=$SONAR_URL \
                        -Dsonar.token=$TOKEN
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME ./app'
            }
        }

        stage('Push to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-registry-user-pass', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login $DOCKER_REGISTRY -u $USER --password-stdin
                    docker tag $IMAGE_NAME $DOCKER_REGISTRY/$IMAGE_NAME
                    docker push $DOCKER_REGISTRY/$IMAGE_NAME
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                docker stop app || true
                docker rm app || true
                docker run -d -p 8083:8080 --name app $DOCKER_REGISTRY/$IMAGE_NAME
                """
            }
        }
    }
}
