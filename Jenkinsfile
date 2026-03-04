pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
        jdk 'JDK-17'
    }

    stages {

        // ─── ÉTAPE 1 : Récupérer le code depuis GitHub ───
        stage('Checkout') {
            steps {
                echo '=== Récupération du code GitHub ==='
                checkout scm
            }
        }

        // ─── ÉTAPE 2 : Builder common-lib EN PREMIER ───
        // obligatoire car tous les services en dépendent
        stage('Build common-lib') {
            steps {
                echo '=== Build common-lib ==='
                dir('common-lib') {
                    bat 'mvn clean install -DskipTests -Dgpg.skip=true'
                }
            }
        }

        // ─── ÉTAPE 3 : Builder tous les services ───
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
                            bat 'mvn clean package -DskipTests'
                        }
                    }
                }
            }
        }

        // ─── ÉTAPE 4 : Builder les images Docker ───
        stage('Docker Build') {
            steps {
                echo '=== Build des images Docker ==='
                bat 'docker-compose build'
            }
        }

        // ─── ÉTAPE 5 : Déployer ───
        stage('Deploy') {
            steps {
                echo '=== Déploiement ==='
                bat 'docker-compose up -d'
            }
        }
    }

    // ─── RÉSULTAT FINAL ───
    post {
        success {
            echo '✅ Pipeline réussi — tout est déployé !'
        }
        failure {
            echo '❌ Pipeline échoué — vérifie les logs !'
        }
    }
}
```

---

## Pourquoi `bat` et pas `sh` ?
```
sh  → Linux / Mac
bat → Windows   ← ton PC est Windows donc bat