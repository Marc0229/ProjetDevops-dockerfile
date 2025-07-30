pipeline {
    agent any

    environment {
        COMPOSE_CMD = "docker-compose"
        DB_CONTAINER = "projetdevops-dockerfile_db_1"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Marc0229/ProjetDevops-dockerfile.git'
            }
        }

        stage('Build Images') {
            steps {
                sh "${COMPOSE_CMD} build"
            }
        }

        stage('Deploy Containers') {
            steps {
                sh "${COMPOSE_CMD} down -v || true"
                sh "${COMPOSE_CMD} up -d"
            }
        }

        stage('Check Services') {
            steps {
                sh "docker ps"
                sh "curl -s -o /dev/null -w '%{http_code}' http://localhost:5000 || true"
                sh "curl -s -o /dev/null -w '%{http_code}' http://localhost:8081 || true"
            }
        }

        stage('Test DB Votes Table') {
            steps {
                sh '''
                echo "🔍 Vérification de la table votes..."
                docker exec -i $DB_CONTAINER \
                psql -U postgres -d votesdb -c "SELECT count(*) FROM votes;" || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Déploiement et test réussis !"
        }
        failure {
            echo "❌ Échec du déploiement ou du test"
        }
    }
}

