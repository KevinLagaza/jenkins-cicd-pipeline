pipeline {
    // agent {
    //     docker {
    //         image 'maven:3.9.6-eclipse-temurin-17-alpine'
    //         args '-v $HOME/.m2:/root/.m2 -v /var/run/docker.sock:/var/run/docker.sock'
    //     }
    // }

    agent any

    environment {
        APP_NAME = 'paymybuddy'
        MAVEN_IMAGE = 'maven:3.9.6-eclipse-temurin-17-alpine'
        DOCKER_IMAGE = 'kevinlagaza/${APP_NAME}'
        DOCKER_TAG = '${BUILD_NUMBER}'
        // SONAR_TOKEN = 'SonarQube'
        // DOCKER_CREDENTIALS = 'dockerhub-credentials'
    }

    stages {

        stage('Build') {
            steps {
                echo '========== BUILD =========='
                sh '''
                    docker run --rm \
                        -v \$(pwd):/app \
                        -v \$HOME/.m2:/root/.m2 \
                        -w /app \
                        ${MAVEN_IMAGE} \
                        mvn clean compile -DskipTests
                '''
                echo '========== FINISHED BUILD =========='
            }
        }

        stage('Unit Tests') {
            steps {
                echo '========== UNIT TESTS =========='
                sh '''
                    docker run --rm \
                        -v \$(pwd):/app \
                        -v \$HOME/.m2:/root/.m2 \
                        -w /app \
                        ${MAVEN_IMAGE} \
                        mvn test -Dtest=*Test
                '''
                echo '========== FINISHED UNIT TESTS =========='
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Integration Tests') {
            steps {
                echo '========== INTEGRATION TESTS =========='
                sh '''
                    docker run --rm \
                        -v \$(pwd):/app \
                        -v \$HOME/.m2:/root/.m2 \
                        -w /app \
                        ${MAVEN_IMAGE} \
                        mvn verify -Dtest=*IT -DfailIfNoTests=false
                '''
                echo '========== FINISHED INTEGRATION TESTS =========='
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/target/failsafe-reports/*.xml'
                }
            }
        }

        stage ('Checkstyle Code Analysis'){
            steps {
                echo '========== CHECKSTYLE ANALYSIS =========='
                sh '''
                    docker run --rm \
                        -v \$(pwd):/app \
                        -v \$HOME/.m2:/root/.m2 \
                        -w /app \
                        ${MAVEN_IMAGE} \
                        mvn checkstyle:checkstyle
                '''
                echo '========== FINISHED CHECKSTYLE ANALYSIS =========='
            }
        }

        stage('SonarQube Analysis') {
            // environment {
            //     scannerHome = tool 'sonar-scanner-8'
            // }
            steps {
                echo '========== SONARQUBE ANALYSIS =========='
                // sh 'mvn sonar:sonar -Dsonar.projectKey=samson-jean -Dsonar.token=jenkins_token -Dsonar.language=java -Dsonar.tests=src/test -Dsonar.sources=src/main/java' 
                // withSonarQubeEnv('sonarqube') {
                // sh '''  
                //     mvn sonar:sonar \
                //         -Dsonar.projectKey=kevin_82_webapp \
                //         -Dsonar.organization=samson-jean \
                //         -Dsonar.projectVersion=1.0 \
                //         -Dsonar.sources=src/java \
                //         -Dsonar.tests=src/test \
                //         -Dsonar.java.binaries=target/classes
                // '''
                // }

                withSonarQubeEnv('sonarqube') {
                    sh '''docker run --rm \
                            -v "$(pwd)":/app \
                            -v "$HOME/.m2":/root/.m2 \
                            -w /app \
                            -e SONAR_HOST_URL="${SONAR_HOST_URL}" \
                            -e SONAR_TOKEN="${SONAR_AUTH_TOKEN}" \
                            ${MAVEN_IMAGE} \
                            mvn sonar:sonar \
                                -Dsonar.projectKey=kevin_82_webapp \
                                -Dsonar.organization=samson-jean \
                                -Dsonar.host.url=${SONAR_HOST_URL} \
                                -Dsonar.token=${SONAR_AUTH_TOKEN}
                    '''
                }
                echo 'FINISHED SONARQUBE ANALYSIS'
            }
        }

        stage('Compilation') {
            steps {
                echo '========== COMPILATION =========='
                sh  '''
                    docker run --rm \
                        -v "$(pwd)":/app \
                        -v "$HOME/.m2":/root/.m2 \
                        -w /app \
                        -e SONAR_HOST_URL="${SONAR_HOST_URL}" \
                        -e SONAR_TOKEN="${SONAR_AUTH_TOKEN}" \
                        ${MAVEN_IMAGE} \
                        mvn package -DskipTests
                '''
                sh 'ls -la target/*.jar'
                echo 'FINISHED COMPILATION'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
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
        // }
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