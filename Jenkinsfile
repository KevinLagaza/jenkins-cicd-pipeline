pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17-alpine'
            args '-v $HOME/.m2:/root/.m2 -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        APP_NAME = 'paymybuddy'
        DOCKER_IMAGE = "kevinlagaza/${APP_NAME}"
        DOCKER_TAG = "${BUILD_NUMBER}"
        // DOCKER_CREDENTIALS = 'dockerhub-credentials'
    }

    stages {
        stage('Build') {
            steps {
                echo "========== BUILD =========="
                sh 'mvn clean compile -DskipTests'
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
        always {
            cleanWs()
        }
    }
}