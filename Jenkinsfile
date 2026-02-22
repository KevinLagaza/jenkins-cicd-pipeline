pipeline {
    agent any

    environment {
        APP_NAME = 'paymybuddy'
        MAVEN_IMAGE = 'maven:3.9.6-eclipse-temurin-17-alpine'
        DOCKER_IMAGE = 'kevinlagaza/${APP_NAME}'
        DOCKER_TAG = '${BUILD_NUMBER}'
        REGISTRY_URL = 'https://index.docker.io/v1/'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        // SONAR_TOKEN = 'SonarQube'
    }

    stages {

        // stage('Checkout') {
        //     steps {
        //         // Clean workspace before checkout
        //         cleanWs()
                
        //         // Explicit checkout
        //         checkout scm
                
        //         // Verify checkout
        //         sh 'ls -la'
        //         sh 'git status'
        //     }
        // }

        stage('Build') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                    reuseNode true
                }
            }
            steps {
                echo '========== BUILD =========='
                sh 'mvn clean compile -DskipTests'
                echo '========== FINISHED BUILD =========='
            }
        }

        // stage('Unit Tests') {
        //     agent {
        //         docker {
        //             image 'maven:3.9.6-eclipse-temurin-17-alpine'
        //             args '-v $HOME/.m2:/root/.m2'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo '========== UNIT TESTS =========='
        //         sh 'mvn test -Dtest=*Test'
        //         echo '========== FINISHED UNIT TESTS =========='
        //     }
        //     post {
        //         always {
        //             junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        //         }
        //     }
        // }

        // stage('Integration Tests') {
        //     agent {
        //         docker {
        //             image 'maven:3.9.6-eclipse-temurin-17-alpine'
        //             args '-v $HOME/.m2:/root/.m2'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo '========== INTEGRATION TESTS =========='
        //         sh 'mvn verify -Dtest=*IT -DfailIfNoTests=false'
        //         echo '========== FINISHED INTEGRATION TESTS =========='
        //     }
        //     post {
        //         always {
        //             junit allowEmptyResults: true, testResults: '**/target/failsafe-reports/*.xml'
        //         }
        //     }
        // }

        stage ('Checkstyle Code Analysis'){
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                    reuseNode true
                }
            }
            steps {
                echo '========== CHECKSTYLE ANALYSIS =========='
                sh 'mvn checkstyle:checkstyle'
                echo '========== FINISHED CHECKSTYLE ANALYSIS =========='
            }
        }

        // stage('SonarQube Analysis') {
        //     agent {
        //         docker {
        //             image 'maven:3.9.6-eclipse-temurin-17-alpine'
        //             args '-v $HOME/.m2:/root/.m2'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo '========== SONARQUBE ANALYSIS =========='
        //         withSonarQubeEnv('sonarqube') {
        //             sh '''
        //                 mvn sonar:sonar \
        //                     -Dsonar.projectKey=kevin_82_webapp \
        //                     -Dsonar.organization=samson-jean \
        //                     -Dsonar.projectVersion=1.0 \
        //                     -Dsonar.java.source=17
        //             '''
        //         }
        //         echo '========== FINISHED SONARQUBE ANALYSIS =========='
        //     }
        // }


        stage('Compilation') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                    reuseNode true
                }
            }
            steps {
                echo '========== COMPILATION =========='
                sh  'mvn package -DskipTests'
                sh 'ls -la target/*.jar'
                echo '========== FINISHED COMPILATION =========='
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:24-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                    reuseNode true
                }
            }
            steps {
                echo '========== BUILD DOCKER IMAGE =========='
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                '''
                echo '========== FINISHED BUILDING DOCKER IMAGE =========='
            }
        }

        // stage('Push to Registry') {
        //     steps {
        //         script {
        //             docker.withRegistry("${REGISTRY_URL}", "${DOCKER_CREDENTIALS}") {
        //                 dockerImage.push("${DOCKER_TAG}")
        //             }
        //         }
        //     }
        // }

        // stage('Build Docker Image') {
        //     agent {
        //         docker {
        //             image 'docker:24-cli'
        //             args '-v /var/run/docker.sock:/var/run/docker.sock'
        //         }
        //     }
        //     steps {
        //         echo '========== BUILD DOCKER IMAGE =========='
        //         sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
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
        //         echo '========== PUSH DOCKER IMAGE =========='
        //         withCredentials([usernamePassword(
        //             credentialsId: DOCKER_CREDENTIALS,
        //             usernameVariable: 'DOCKER_USER',
        //             passwordVariable: 'DOCKER_PASS'
        //         )]) {
        //             sh '''
        //                 echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
        //                 docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
        //             '''
        //         }
        //     }
        // }


        // stage('Deploy to Production') {
        //     when {
        //         branch 'main'
        //     }
        //     input {
        //         message 'Deploy to production?'
        //         ok 'Deploy'
        //     }
        //     steps {
        //         echo '========== DEPLOY TO PRODUCTION =========='
        //         sshagent(['ssh-production-key']) {
        //             sh '''
        //                 ssh -o StrictHostKeyChecking=no user@production-server '
        //                     docker pull ${DOCKER_IMAGE}:${DOCKER_TAG} &&
        //                     docker stop ${APP_NAME} || true &&
        //                     docker rm ${APP_NAME} || true &&
        //                     docker run -d --name ${APP_NAME} -p 8080:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}
        //                 '
        //             '''
        //         }
        //     }
        }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
        always {
            cleanWs()
        }
    }
}