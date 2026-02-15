pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17-alpine'
            // args '-v $HOME/.m2:/root/.m2 -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        APP_NAME = 'paymybuddy'
        DOCKER_IMAGE = "kevinlagaza/${APP_NAME}"
        DOCKER_TAG = "${BUILD_NUMBER}"
        // DOCKER_CREDENTIALS = 'dockerhub-credentials'
    }

    stages {

        stage('Unit Tests') {
            steps {
                echo "========== UNIT TESTS =========="
                sh 'mvn test -Dtest=*Test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                    jacoco(
                        execPattern: '**/target/jacoco.exec',
                        classPattern: '**/target/classes',
                        sourcePattern: '**/src/main/java',
                        exclusionPattern: '**/test/**'
                    )
                }
            }
        }

        stage('Integration Tests') {
            steps {
                echo "========== INTEGRATION TESTS =========="
                sh 'mvn verify -Dtest=*IT -DfailIfNoTests=false'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/target/failsafe-reports/*.xml'
                }
            }
        }

        stage('Build') {
            steps {
                echo "========== BUILD =========="
                sh 'mvn clean install'
                // sh 'mvn clean compile -DskipTests'
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