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

        stage('SonarQube Analysis') {
            steps {
                echo "========== SONARQUBE ANALYSIS =========="
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build') {
            steps {
                echo "========== BUILD =========="
                sh 'mvn Package -DskipTests'
                sh 'ls -la target/*.jar'
                // sh 'mvn clean compile -DskipTests'
            }
        }

        // stage('Build Docker Image') {
        //     agent {
        //         docker {
        //             image 'docker:24-cli'
        //             args '-v /var/run/docker.sock:/var/run/docker.sock'
        //         }
        //     }
        //     steps {
        //         echo "========== BUILD DOCKER IMAGE =========="
        //         sh """
        //             docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
        //         """
        //     }
        // }

        // stage('Push Docker Image') {
        //     agent {
        //         docker {
        //             image 'docker:24-cli'
        //             args '-v /var/run/docker.sock:/var/run/docker.sock'
        //         }
        //     }
        //     steps {
        //         echo "========== PUSH DOCKER IMAGE =========="
        //         withCredentials([usernamePassword(
        //             credentialsId: DOCKER_CREDENTIALS,
        //             usernameVariable: 'DOCKER_USER',
        //             passwordVariable: 'DOCKER_PASS'
        //         )]) {
        //             sh """
        //                 echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
        //                 docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
        //             """
        //         }
        //     }
        // }
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