pipeline {
agent any

```
tools {
    maven 'maven3'
}

environment {
    SONAR_HOST = 'http://98.91.191.208:9000'
    NEXUS_URL = '98.91.191.208:8082'
}

stages {

    stage('Checkout') {
        steps {
            git branch: 'main',
            url: 'https://github.com/Harshith078/ci-cd-project.git'
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
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.host.url=$SONAR_HOST \
                    -Dsonar.login=$TOKEN
                    '''
                }
            }
        }
    }

    stage('Build + Push + Deploy (EC2-2)') {
        steps {
            sshagent(['ec2-docker-ssh']) {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@32.192.187.102 '

                # Clean old code
                rm -rf app
                git clone https://github.com/Harshith078/ci-cd-project.git app

                cd app/app

                # Build JAR
                mvn clean package

                # Build Docker image
                docker build -t demo-app .

                # Tag for Nexus
                docker tag demo-app 98.91.191.208:8082/demo-app

                # Login to Nexus
                docker login 98.91.191.208:8082 -u jenkins -p Manualchanges@123

                # Push image
                docker push 98.91.191.208:8082/demo-app

                # Deploy container
                docker rm -f app || true
                docker run -d -p 8083:8080 --name app 98.91.191.208:8082/demo-app

                '
                '''
            }
        }
    }
}
```

}
