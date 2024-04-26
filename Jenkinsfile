def microservices = ['ecomm-cart','ecomm-order','ecomm-product','ecomm-web']

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout the main repository
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: '*']], 
                    userRemoteConfigs: [[url: 'https://github.com/youssefrmili/ecom.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                script {
                    // Iterate over each microservice folder
                    for (def service in microservices) {
                        // Navigate into the microservice folder
                        dir(service) {
                            // Build the microservice
                            sh 'mvn clean install'
                        }
                    }
                }
            }
        }

        stage('Unit Test') {
            steps {
                script {
                    // Iterate over each microservice folder
                    for (def service in microservices) {
                        // Navigate into the microservice folder
                        dir(service) {
                            // Test the microservice
                            sh 'mvn test'
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Iterate over each microservice folder
                    for (def service in microservices) {
                        // Navigate into the microservice folder
                        dir(service) {
                            // Execute SAST with SonarQube
                            withSonarQubeEnv(credentialsId: 'sonarqube-id') {
                                sh 'mvn sonar:sonar'
                                sh 'cat target/sonar/report-task.txt'
                            }
                        }
                    }
                }
            }
        }

        stage('Build and Push Docker Images') {
            environment {
                DOCKER_CREDENTIALS = credentials('dockerhub')
            }
            steps {
                script {
                    // Docker login
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }

                    // Iterate over each microservice folder
                    for (def service in microservices) {
                        // Navigate into the microservice folder
                        dir(service) {
                            // Build Docker image
                            sh "docker build -t youssefrm/${service}:latest ."
                            // Push Docker image
                            sh "docker push youssefrm/${service}:latest"
                        }
                    }
                }
            }
        }
    }
}
