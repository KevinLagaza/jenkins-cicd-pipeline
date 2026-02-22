pipeline {
    agent any

    environment {
        APP_NAME = 'paymybuddy'
        MAVEN_IMAGE = 'maven:3.9.6-eclipse-temurin-17-alpine'
        DOCKER_IMAGE = 'kevinlagaza/${APP_NAME}'
        DOCKER_TAG = '${BUILD_NUMBER}'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        // Deployment
        STAGING_HOST = credentials('staging-host') 
        PRODUCTION_HOST = credentials('production-host')
        STAGING_SSH_KEY = 'staging-ssh-key'
        PRODUCTION_SSH_KEY = 'production-ssh-key'
        CONTAINER_PORT = '8080'
        APP_PORT = '8080'
    }

    stages {

        // stage('Unit Tests') {
        //     agent {
        //         docker {
        //             image "${MAVEN_IMAGE}"
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
        //             image "${MAVEN_IMAGE}"
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

        // stage ('Checkstyle Code Analysis'){
        //     agent {
        //         docker {
        //             image "${MAVEN_IMAGE}"
        //             args '-v $HOME/.m2:/root/.m2'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo '========== CHECKSTYLE ANALYSIS =========='
        //         sh 'mvn checkstyle:checkstyle'
        //         echo '========== FINISHED CHECKSTYLE ANALYSIS =========='
        //     }
        // }

        // stage('SonarQube Analysis') {
        //     agent {
        //         docker {
        //             image "${MAVEN_IMAGE}"
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

        // stage('Build') {
        //     agent {
        //         docker {
        //             image "${MAVEN_IMAGE}"
        //             args '-v $HOME/.m2:/root/.m2'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo '========== BUILD =========='
        //         sh 'mvn clean compile -DskipTests'
        //         echo '========== FINISHED BUILD =========='
        //     }
        // }

        // stage('Compilation') {
        //     agent {
        //         docker {
        //             image "${MAVEN_IMAGE}"
        //             args '-v $HOME/.m2:/root/.m2'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo '========== COMPILATION =========='
        //         sh  'mvn package -DskipTests'
        //         sh 'ls -la target/*.jar'
        //         echo '========== FINISHED COMPILATION =========='
        //     }
        //     post {
        //         success {
        //             archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        //         }
        //     }
        // }

        // stage('Build Docker Image') {
        //     agent {
        //         docker {
        //             image 'docker:24-cli'
        //             args '-v /var/run/docker.sock:/var/run/docker.sock'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo '========== BUILD DOCKER IMAGE =========='
        //         sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
        //         echo '========== FINISHED BUILDING DOCKER IMAGE =========='
        //     }
        // }

        // stage('Push Docker Image') {
        //     agent {
        //         docker {
        //             image 'docker:24-cli'
        //             args '-v /var/run/docker.sock:/var/run/docker.sock'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo "========== PUSH DOCKER IMAGE =========="
        //         withCredentials([usernamePassword(
        //             credentialsId: "${DOCKER_CREDENTIALS}",
        //             usernameVariable: 'DOCKER_USER',
        //             passwordVariable: 'DOCKER_PASS'
        //         )]) {
        //             sh """
        //                 echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
        //                 docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
        //                 docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true
        //                 docker logout
        //             """
        //         }
        //         echo "========== FINISHED PUSHING DOCKER IMAGE =========="
        //     }
        // }

        // stage('Cleanup') {
        //     steps {
        //         sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
        //     }
        // }

        stage('Deploy to Staging') {

            // steps {
            //     echo "GIT_BRANCH: ${env.GIT_BRANCH}"
            //     echo "BRANCH_NAME: ${env.BRANCH_NAME}"
            //     echo "GIT_LOCAL_BRANCH: ${env.GIT_LOCAL_BRANCH}"
            //     sh 'git branch --show-current || echo "Cannot determine branch"'
            //     sh 'git rev-parse --abbrev-ref HEAD || echo "Cannot get HEAD"'
            // }

            when {
                expression { 
                    return env.GIT_BRANCH == 'origin/main'
                }
            }
            steps {
                echo "========== DEPLOY TO STAGING =========="
                sshagent(credentials: ["${STAGING_SSH_KEY}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${STAGING_HOST} '
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG} &&
                            docker stop ${APP_NAME} || true &&
                            docker rm ${APP_NAME} || true &&
                            docker run -d \
                                --name ${APP_NAME} \
                                -p ${APP_PORT}:${CONTAINER_PORT} \
                                --restart unless-stopped \
                                -e SPRING_PROFILES_ACTIVE=staging \
                                ${DOCKER_IMAGE}:${DOCKER_TAG} &&
                            sleep 10 &&
                            docker ps | grep ${APP_NAME}
                        '
                    """
                }
            }
            post {
                success {
                    echo "✅ Staging deployment successful!"
                }
                failure {
                    echo "❌ Staging deployment failed!"
                }
            }
        }


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