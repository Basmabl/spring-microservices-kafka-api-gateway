pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
        jdk 'JDK-17'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '=== Récupération du code GitHub ==='
                checkout scm
            }
        }

        stage('Build common-lib') {
            steps {
                echo '=== Build common-lib ==='
                dir('common-lib') {
                    sh 'mvn clean install -DskipTests -Dgpg.skip=true'
                }
            }
        }

        stage('Build Services') {
            steps {
                script {
                    def services = [
                        'service-registry',
                        'api-gateway',
                        'identity-service',
                        'order-service',
                        'payment-service',
                        'product-service',
                        'email-service'
                    ]
                    services.each { svc ->
                        echo "=== Build: ${svc} ==="
                        dir(svc) {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def services = [
                        'identity-service',
                        'order-service',
                        'payment-service',
                        'product-service',
                        'email-service'
                    ]
                    withSonarQubeEnv('SonarQube') {
                        services.each { svc ->
                            echo "=== SonarQube: ${svc} ==="
                            dir(svc) {
                                sh "mvn sonar:sonar -Dsonar.projectKey=${svc} -Dsonar.host.url=http://host.docker.internal:9000"
                            }
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo '=== Build des images Docker ==='
                sh 'docker-compose -p myapp build'
            }
        }

        stage('Deploy') {
            steps {
                echo '=== Déploiement ==='
                sh 'docker-compose -p myapp down --remove-orphans || true'
                sh 'docker-compose -p myapp up -d'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline réussi !'
        }
        failure {
            echo '❌ Pipeline échoué !'
        }
    }
}