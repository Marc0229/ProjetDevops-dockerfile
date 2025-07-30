pipeline {
    agent any

    environment {
        COMPOSE_CMD = "docker-compose"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Marc0229/ProjetDevops-dockerfile.git'
            }
        }

        stage('Clean Old Containers') {
            steps {
                echo "🧹 Suppression des anciens conteneurs..."
                sh 'docker-compose down -v || true'
            }
        }

        stage('Build Images') {
            steps {
                echo "🔨 Build des images Docker..."
                sh "${COMPOSE_CMD} build"
            }
        }

        stage('Deploy Containers') {
            steps {
                echo "🚀 Lancement des conteneurs..."
                sh "${COMPOSE_CMD} up -d"
            }
        }

        stage('Check Running Containers') {
            steps {
                echo "🔍 Vérification des conteneurs en cours d'exécution..."
                sh "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"
            }
        }

        stage('Check DB Container') {
            steps {
                script {
                    echo "🔍 Recherche du conteneur PostgreSQL..."
                    def dbContainer = sh(
                        script: "docker ps --filter 'ancestor=postgres:13-alpine' --format '{{.Names}}'",
                        returnStdout: true
                    ).trim()

                    if (dbContainer) {
                        echo "✅ Conteneur PostgreSQL trouvé : ${dbContainer}"
                    } else {
                        error "❌ Aucun conteneur PostgreSQL trouvé !"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Déploiement terminé avec succès !"
        }
        failure {
            echo "❌ Le pipeline a échoué."
        }
    }
}

