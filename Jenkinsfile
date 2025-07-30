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
                echo "🔍 Vérification des conteneurs..."
                sh "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"
            }
        }

    post {
        success {
            echo "✅ Déploiement et tests réussis !"
        }
        failure {
            echo "❌ Le pipeline a échoué."
        }
    }
}

