pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'hemanth251/simple-app:latest'
        SONAR_HOST_URL = 'http://localhost:9000'
    }

    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()
                git url: 'https://github.com/heroku/java-getting-started.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                sh "./mvnw sonar:sonar -Dsonar.projectKey=simple-app -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN"
            }
        }

        stage('Docker Build & Push') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                
                sh """
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                docker build -t hemanth251/simple-app:latest . -f ./Dockerfile
                docker push hemanth251/simple-app:latest
                """
            }
        }
    }
}


        stage('Deploy') {
            steps {
                sh """
                docker stop simple-app || true
                docker rm simple-app || true
                docker run --rm -d --name simple-app -p 8080:8080 $DOCKER_IMAGE
                """
            }
        }
    }
}
