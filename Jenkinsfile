pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKER_IMAGE = "senthilkumarsoundararajan/java-maven-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        SONAR_HOST = "http://44.201.181.219:9000"
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }
        stage('Maven Build') {
            steps {
                echo 'Building with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=java-maven-app'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }
        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE:$DOCKER_TAG
                        docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_IMAGE:latest
                        docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }
    }
    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
