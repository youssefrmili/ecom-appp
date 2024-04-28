def microservices = ['ecomm-cart']

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

        stage('Check-Git-Secrets') {
            steps {
                // Iterate over each microservice folder
                for (def service in microservices) {
                    // Navigate into the microservice folder
                    dir(service) {
                        sh 'rm trufflehog || true'
                        sh 'docker run gesellix/trufflehog --json https://github.com/youssefrmili/Ecommerce-APP.git > trufflehog'
                        sh 'cat trufflehog'
                    }
                }
            }
        }

        stage('Source Composition Analysis') {
            steps {
                // Iterate over each microservice folder
                for (def service in microservices) {
                    // Navigate into the microservice folder
                    dir(service) {
                        sh 'rm owasp* || true'
                        sh 'wget "https://raw.githubusercontent.com/youssefrmili/Ecommerce-APP/test/owasp-dependency-check.sh" '
                        sh 'chmod +x owasp-dependency-check.sh'
                        sh 'bash owasp-dependency-check.sh'
                        sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
                    }
                }
            }
        }

        stage('Build') {
            steps {
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

        stage('Unit Test') {
            steps {
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

        stage('SonarQube Analysis') {
            steps {
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

        stage('Quality Gate') {
            steps {
                // Quality Gate
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
            }
        }

        stage('Docker Login') {
            steps {
                // Docker login
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                }
            }
        }

        stage('Docker Build') {
            steps {
                // Iterate over each microservice folder
                for (def service in microservices) {
                    // Navigate into the microservice folder
                    dir(service) {
                        // Build Docker image
                        sh "docker build -t youssefrm/${service}:latest ."
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                // Iterate over each microservice folder
                for (def service in microservices) {
                    // Scan the Docker image using Trivy
                    sh "docker run --rm -v /home/youssef/.cache:/root/.cache/ aquasec/trivy image --scanners vuln  youssefrm/ecomm-product:latest > trivy.txt"
                }
            }
        }

        stage('Docker Push') {
            steps {
                // Iterate over each microservice folder
                for (def service in microservices) {
                    // Push Docker image
                    sh "docker push youssefrm/${service}:latest"
                }
            }
        }
    }
}

