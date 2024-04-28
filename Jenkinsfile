def microservices = ['ecomm-cart']

def iterateOverMicroservices(Closure body) {
    for (def service in microservices) {
        dir(service) {
            body()
        }
    }
}

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
                script {
                    iterateOverMicroservices {
                        sh 'rm trufflehog || true'
                        sh 'docker run gesellix/trufflehog --json https://github.com/youssefrmili/Ecommerce-APP.git > trufflehog'
                        sh 'cat trufflehog'
                    }
                }
            }
        }

        stage('Source Composition Analysis') {
            steps {
                script {
                    iterateOverMicroservices {
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
                script {
                    iterateOverMicroservices {
                        sh 'mvn clean install'
                    }
                }
            }
        }

        stage('Unit Test') {
            steps {
                script {
                    iterateOverMicroservices {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    iterateOverMicroservices {
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
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-id'
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    // Docker login
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    iterateOverMicroservices {
                        sh "docker build -t youssefrm/${service}:latest ."
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    iterateOverMicroservices {
                        sh "docker run --rm -v /home/youssef/.cache:/root/.cache/ aquasec/trivy image --scanners vuln  youssefrm/ecomm-product:latest > trivy.txt"
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    iterateOverMicroservices {
                        sh "docker push youssefrm/${service}:latest"
                    }
                }
            }
        }
    }
}
