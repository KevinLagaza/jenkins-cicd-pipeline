pipeline {
    agent any

    environment {
        APP_NAME = 'paymybuddy'
        MAVEN_IMAGE = 'maven:3.9.6-eclipse-temurin-17-alpine'
        DOCKER_IMAGE = 'kevinlagaza/${APP_NAME}'
        DOCKER_TAG = '${BUILD_NUMBER}'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'

        // AWS servers
        STAGING_HOST = credentials('staging-host') 
        PRODUCTION_HOST = credentials('production-host')

        // SSH Configuration
        SSH_USER = 'ubuntu'
        STAGING_SSH_KEY = 'staging-ssh-key'
        PRODUCTION_SSH_KEY = 'production-ssh-key'

        // Application Configuration
        CONTAINER_PORT = '8080'
        APP_PORT = '8080'

        // Database Configuration
        DB_CONTAINER_NAME = 'mysql-paymybuddy'
        DB_PORT = '3306'
        DB_ROOT_PASSWORD = 'password'
        DB_NAME = 'paymybuddy'
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

        stage('Build') {
            agent {
                docker {
                    image "${MAVEN_IMAGE}"
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

        stage('Compilation') {
            agent {
                docker {
                    image "${MAVEN_IMAGE}"
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
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                echo '========== FINISHED BUILDING DOCKER IMAGE =========='
            }
        }

        stage('Push Docker Image') {
            agent {
                docker {
                    image 'docker:24-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                    reuseNode true
                }
            }
            steps {
                echo "========== PUSH DOCKER IMAGE =========="
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                        docker logout
                    """
                }
                echo "========== FINISHED PUSHING DOCKER IMAGE =========="
            }
        }

        stage('Deploy to Staging') {

            agent {
                docker {
                    image 'docker:24-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                    reuseNode true
                }
            }

            when {
                expression { 
                    return env.GIT_BRANCH == 'origin/main'
                }
            }

            steps {
                echo "========== DEPLOY TO STAGING =========="
                sshagent(credentials: ["${STAGING_SSH_KEY}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${STAGING_HOST} "

                            echo '=== Checking existing MySQL container ===' &&
                            if docker ps -a | grep -q ${DB_CONTAINER_NAME}; then
                                echo 'MySQL container exists, checking if running...' &&
                                if ! docker ps | grep -q ${DB_CONTAINER_NAME}; then
                                    echo 'Starting existing MySQL container...' &&
                                    docker start ${DB_CONTAINER_NAME}
                                else
                                    echo 'MySQL container already running'
                                fi
                            else
                                echo '=== Starting MySQL container ===' &&
                                docker run -d \
                                    --name ${DB_CONTAINER_NAME} \
                                    -p ${DB_PORT}:3306 \
                                    -e MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD} \
                                    -e MYSQL_DATABASE=${DB_NAME} \
                                    mysql:8.0 &&
                                echo 'Waiting for MySQL to be ready...' &&
                                sleep 30
                            fi &&

                            echo '=== Getting Docker bridge IP ===' &&
                            DOCKER_IP=\\\$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${DB_CONTAINER_NAME}) &&
                            echo \"MySQL IP: \\\$DOCKER_IP\" &&

                            echo '=== Waiting for MySQL to accept connections ===' &&
                            for i in 1 2 3 4 5 6 7 8 9 10; do
                                if docker exec ${DB_CONTAINER_NAME} mysql -u root -p${DB_ROOT_PASSWORD} -e 'SELECT 1' 2>/dev/null; then
                                    echo 'MySQL is ready!'
                                    break
                                fi
                                echo \"Waiting for MySQL... attempt \\\$i/10\"
                                sleep 5
                            done &&
                            
                            echo '=== Executing SQL scripts ===' &&
                            docker exec -i ${DB_CONTAINER_NAME} mysql -u root -p${DB_ROOT_PASSWORD} < /tmp/create.sql &&
                            docker exec -i ${DB_CONTAINER_NAME} mysql -u root -p${DB_ROOT_PASSWORD} ${DB_NAME} < /tmp/data.sql &&

                            echo "=== Pulling new image ===" &&
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG} &&

                            echo "=== Stopping old container ===" &&
                            docker stop ${APP_NAME} || true &&
                            docker rm ${APP_NAME} || true &&

                            echo "=== Starting new container ===" &&
                            docker run -d \
                                --name ${APP_NAME} \
                                -p ${APP_PORT}:${CONTAINER_PORT} \
                                -e SPRING_PROFILES_ACTIVE=staging \
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                        "
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
        //         expression { 
        //             return env.GIT_BRANCH == 'origin/main'
        //         }
        //     }
        //     steps {
        //         echo "========== DEPLOY TO PRODUCTION =========="
        //         sshagent(credentials: ["${PRODUCTION_SSH_KEY}"]) {
        //             sh """
        //                 ssh -o StrictHostKeyChecking=no ${SSH_USER}@${PRODUCTION_HOST} "

        //                     echo "=== Pulling new image ===" &&
        //                     docker pull ${DOCKER_IMAGE}:${DOCKER_TAG} &&

        //                     echo "=== Stopping old container ===" &&
        //                     docker stop ${APP_NAME} || true &&
        //                     docker rm ${APP_NAME} || true &&

        //                     echo "=== Starting new container ===" &&
        //                     docker run -d \
        //                         --name ${APP_NAME} \
        //                         -p ${APP_PORT}:${CONTAINER_PORT} \
        //                         -e SPRING_PROFILES_ACTIVE=production \
        //                         ${DOCKER_IMAGE}:${DOCKER_TAG} &&
        //                     sleep 10 &&
        //                     docker ps
        //                 "
        //             """
        //         }
        //     }
        //     post {
        //         success {
        //             echo "✅ Production deployment successful!"
        //         }
        //         failure {
        //             echo "❌ Production deployment failed!"
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