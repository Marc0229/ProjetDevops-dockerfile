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
	
	stage('SonarQube Analysis') {
	    steps {
		withSonarQubeEnv('sonarqube') {
		    withSonarQubeScanner('sonar-scanner') {
		        sh '''
		        sonar-scanner \
		          -Dsonar.projectKey=projetdevops \
		          -Dsonar.sources=. \
		          -Dsonar.host.url=http://localhost:9000
		        '''
		    }
		}
	    }
	}

	
        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin

                    docker push ${DOCKER_USER}/projetdevops-dockerfile:vote-latest
                    docker push ${DOCKER_USER}/projetdevops-dockerfile:worker-latest
                    docker push ${DOCKER_USER}/projetdevops-dockerfile:result-latest
                    '''
                }
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
            emailext(
                subject: "✅ Succès du Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a réussi !\nVoir les détails : ${env.BUILD_URL}",
                to: "marcorelabdel@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ Échec du Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué.\nVoir les logs : ${env.BUILD_URL}",
                to: "marcorelabdel@gmail.com"
            )
        }
    }
}


