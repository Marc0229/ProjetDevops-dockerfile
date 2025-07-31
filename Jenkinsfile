pipeline {
    agent any

    environment {
        COMPOSE_CMD = "docker-compose"
        DOCKERHUB_USERNAME = credentials('dockerhub-credentials').usr
        DOCKERHUB_PASSWORD = credentials('dockerhub-credentials').psw
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

        stage('Push Docker Images') {
            steps {
                echo "📤 Connexion et Push vers Docker Hub..."
                sh '''
                echo "${DOCKERHUB_PASSWORD}" | docker login -u "${DOCKERHUB_USERNAME}" --password-stdin

                docker push ${DOCKERHUB_USERNAME}/projetdevops-dockerfile:vote-latest
                docker push ${DOCKERHUB_USERNAME}/projetdevops-dockerfile:worker-latest
                docker push ${DOCKERHUB_USERNAME}/projetdevops-dockerfile:result-latest
                '''
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
            emailext (
                subject: "✅ Succès du Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a réussi !\nVoir les détails : ${env.BUILD_URL}",
                to: "marcorelabdel@gmail.com"
            )
        }
        failure {
            emailext (
                subject: "❌ Échec du Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué.\nVoir les logs : ${env.BUILD_URL}",
                to: "marcorelabdel@gmail.com"
            )
        }
    }
}

