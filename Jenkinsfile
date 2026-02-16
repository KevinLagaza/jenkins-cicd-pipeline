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
        SONAR_TOKEN = 'SonarQube'
        // DOCKER_CREDENTIALS = 'dockerhub-credentials'
    }

    stages {

        // stage('Unit Tests') {
        //     steps {
        //         echo "========== UNIT TESTS =========="
        //         sh 'mvn test -Dtest=*Test'
        //     }
        //     post {
        //         always {
        //             junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        //             jacoco(
        //                 execPattern: '**/target/jacoco.exec',
        //                 classPattern: '**/target/classes',
        //                 sourcePattern: '**/src/main/java',
        //                 exclusionPattern: '**/test/**'
        //             )
        //         }
        //     }
        // }

        // stage('Integration Tests') {
        //     steps {
        //         echo "========== INTEGRATION TESTS =========="
        //         sh 'mvn verify -Dtest=*IT -DfailIfNoTests=false'
        //     }
        //     post {
        //         always {
        //             junit allowEmptyResults: true, testResults: '**/target/failsafe-reports/*.xml'
        //         }
        //     }
        // }

        // stage ('checkstyle code analysis'){
        //     steps {
        //         echo "========== checkstyle ANALYSIS =========="
        //         sh 'mvn checkstyle:checkstyle'
        //     }
        // }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonar-scanner-8'
            }
            steps {
                echo "========== SONARQUBE ANALYSIS =========="
                // sh 'mvn sonar:sonar -Dsonar.projectKey=samson-jean -Dsonar.token=jenkins_token -Dsonar.language=java -Dsonar.tests=src/test -Dsonar.sources=src/main/java' 
                withSonarQubeEnv('sonar-server') {
                sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=samson-jean \
                        -Dsonar.organization=samson_jean \
                //      -Dsonar.projectName=javaapp-repo \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/java \
                        -Dsonar.tests=src/test \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                   '''
                }
            }
        }

        // stage('Build') {
        //     steps {
        //         echo "========== BUILD =========="
        //         sh 'mvn Package -DskipTests'
        //         sh 'ls -la target/*.jar'
        //         // sh 'mvn clean compile -DskipTests'
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


        // stage('Deploy to Production') {
        //     when {
        //         branch 'main'
        //     }
        //     input {
        //         message "Deploy to production?"
        //         ok "Deploy"
        //     }
        //     steps {
        //         echo "========== DEPLOY TO PRODUCTION =========="
        //         sshagent(['ssh-production-key']) {
        //             sh """
        //                 ssh -o StrictHostKeyChecking=no user@production-server '
        //                     docker pull ${DOCKER_IMAGE}:${DOCKER_TAG} &&
        //                     docker stop ${APP_NAME} || true &&
        //                     docker rm ${APP_NAME} || true &&
        //                     docker run -d --name ${APP_NAME} -p 8080:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}
        //                 '
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