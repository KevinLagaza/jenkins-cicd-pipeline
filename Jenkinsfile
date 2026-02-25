@Library('shared-library-jenkins')_

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
        DB_ROOT_PASSWORD = credentials('db-root-pwd')
        DB_NAME = 'paymybuddy'

        // Sonarqube
        SONAR_PROJECT_KEY = credentials('sonar-project-key')
        SONAR_ORGANIZATION = credentials('sonar-organization')
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
        //             sh """
        //                 mvn sonar:sonar \
        //                     -Dsonar.projectKey=${PROJECT_KEY} \
        //                     -Dsonar.organization=${SONAR_ORGANIZATION} \
        //                     -Dsonar.projectVersion=1.0 \
        //                     -Dsonar.java.source=17
        //             """
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

            when {
                expression { 
                    return env.GIT_BRANCH == 'origin/develop'
                }
            }

            steps {
                deployApp(
                        environment: 'staging',
                        host: "${STAGING_HOST}",
                        sshCredentialId: "${STAGING_SSH_KEY}",
                        springProfile: 'staging'
                    )
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

        stage('Test in staging') {

            when {
                expression { 
                    return env.GIT_BRANCH == 'origin/develop'
                }
            }

            steps {
                testEnvironment(
                    host: "${STAGING_HOST}",
                    sshCredentialId: "${STAGING_SSH_KEY}"
                )
            }
        }

        stage('Deploy to Production') {

            when {
                expression { 
                    return env.GIT_BRANCH == 'origin/develop'
                }
            }

            steps {
                deployApp(
                        environment: 'production',
                        host: "${PRODUCTION_HOST}",
                        sshCredentialId: "${PRODUCTION_SSH_KEY}",
                        springProfile: 'production'
                )
            }

            post {
                success {
                    echo "✅ Production deployment successful!"
                }
                failure {
                    echo "❌ Production deployment failed!"
                }
            }
        }

        stage('Test in Production') {

            when {
                expression { 
                    return env.GIT_BRANCH == 'origin/develop'
                }
            }

            steps {
                testEnvironment(
                    host: "${PRODUCTION_HOST}",
                    sshCredentialId: "${PRODUCTION_SSH_KEY}"
                )
            }
        }

    }

    post {
        success {
            echo "✅ Pipeline execution successful!"
            script {
                slackNotifier(currentBuild.result)
            }
        }
        failure {
            echo "❌ Pipeline execution failed!"
            script {
                slackNotifier(currentBuild.result)
            }
        }
        always {
            cleanWs()
        }
    }
}