pipeline {
    agent any

    environment {
        APP_IP = "<EC2-2-IP>"
        IMAGE = "app-image"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/<your-username>/<repo>.git'
            }
        }

        stage('Build') {
            steps {
                dir('app') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('SonarQube') {
            steps {
                dir('app') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'TOKEN')]) {
                        sh """
                        mvn sonar:sonar \
                        -Dsonar.host.url=http://$APP_IP:9000 \
                        -Dsonar.token=$TOKEN
                        """
                    }
                }
            }
        }

        stage('Build Docker') {
            steps {
                sh 'docker build -t $IMAGE ./app'
            }
        }

        stage('Push to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-registry-user-pass', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login $APP_IP:8082 -u $USER --password-stdin
                    docker tag $IMAGE $APP_IP:8082/$IMAGE
                    docker push $APP_IP:8082/$IMAGE
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                ssh ubuntu@$APP_IP '
                  docker stop app || true
                  docker rm app || true
                  docker pull $APP_IP:8082/$IMAGE
                  docker run -d -p 8083:8080 --name app $APP_IP:8082/$IMAGE
                '
                """
            }
        }
    }
}
