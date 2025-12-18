pipeline {
    agent any

    environment {
        // Nexus credentials
        NEXUS_CREDENTIALS = credentials('nexus-admin-credentials')
        DOCKER_CREDENTIALS = credentials ('docker-hub-credentials')
        RELEASE_VERSION = "1.0"
        registry = "malekzahmoul20971/gestion-station-ski"
        registryCredential = 'docker-hub-credentials'
        dockerImage = ''
        IMAGE_TAG = "${RELEASE_VERSION}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'malekzahmoul-5arctic5', url: 'https://github.com/malieo1/5ARCTIC5-GestionSkieur.git'
            }
        }
        stage('Clean') {
            steps {
                echo 'Cleaning the workspace...'
                sh 'mvn clean'
            }
        }
       stage('Test') {
            steps {
                sh 'mvn test '
            }
        }


        stage('Build') {
            steps {
                sh 'mvn install -Dmaven.test.skip=true'
            }
        }

       stage('SonarQube Analysis') {
           steps {
               withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                   sh '''
                       mvn sonar:sonar \
                       -Dsonar.login=$SONAR_TOKEN \
                       -Dsonar.ws.timeout=120 \
                       -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                   '''
               }
           }
       }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-admin-credentials', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                    sh "mvn deploy -Dmaven.test.skip=true"
                }
            }
        }

        stage('Building our image') {
            steps {
                script {
                    dockerImage = docker.build "${registry}:${IMAGE_TAG}"

                }
            }
        }
        stage('Push to DockerHub') {
           steps {
               script {
                   withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                       sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                       sh "docker push ${dockerImage.imageName()}"
                   }
               }
           }
        }
        stage('Deploy with Docker Compose') {
            steps {
                script {
                    // Stop existing containers
                    sh 'docker compose down || true'

                    // Start the applications
                    sh 'docker compose up -d'

                    // Wait for services to initialize
                    sh 'sleep 30'

                    // Verify deployment
                    sh 'docker compose ps'
                }
            }
        }
    }

    post {
        success {
            echo 'Build finished successfully!'
            mail to: 'malekzahmoul20@gmail.com',
                 subject: "Jenkins Job Successful: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                 body: "Good news malek! The job ${env.JOB_NAME} [${env.BUILD_NUMBER}] has finished successfully."
        }
        failure {
            echo 'Build failed!'
            mail to: 'malekzahmoul20@gmail.com',
                 subject: "Jenkins Job Failed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                 body: "Sorry malek, the job ${env.JOB_NAME} [${env.BUILD_NUMBER}] has failed. Please check the Jenkins console output for details."
        }
    }
}
