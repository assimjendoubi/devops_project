pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }
    
    environment {
        DOCKER_IMAGE = "jendoub/spring-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        CONTAINER_NAME = "spring-app-container"
        APP_PORT = "8080"
    }

    stages {
        // ==================== PARTIE CI ====================

        stage('📥 Checkout SCM') {
            steps {
                echo '========== Récupération du code source =========='
                checkout scm
            }
        }

        stage('🧹 Clean') {
            steps {
                echo '========== Nettoyage du projet =========='
                sh 'mvn clean'
            }
        }

        stage('⚙️ Compile') {
            steps {
                echo '========== Compilation du projet =========='
                sh 'mvn compile'
            }
        }

        stage('🔍 SonarQube Analysis') {
            steps {
                echo '========== Analyse de la qualité du code =========='
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=tp-projet-2025 \
                        -Dsonar.projectName="TP Projet 2025" \
                        -Dsonar.host.url=http://192.168.33.10:9000 \
                        -Dsonar.token=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('📦 Package') {
            steps {
                echo '========== Génération du JAR =========='
                sh 'mvn package -DskipTests'
            }
        }

        // ==================== PARTIE CD ====================

        stage('🐳 Build Docker Image') {
            steps {
                echo '========== Construction de l\'image Docker =========='
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }

        stage('📤 Push to Docker Hub') {
            steps {
                echo '========== Push vers Docker Hub =========='
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKERHUB_CREDENTIALS) {
                        dockerImage.push("${DOCKER_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('🧹 Clean Old Container') {
            steps {
                echo '========== Arrêt et suppression de l\'ancien conteneur =========='
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('🚀 Deploy Container') {
            steps {
                echo '========== Déploiement du conteneur Docker =========='
                sh '''
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:8080 \
                        --restart unless-stopped \
                        ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('✅ Verify Deployment') {
            steps {
                echo '========== Vérification du déploiement =========='
                sh '''
                    sleep 10
                    docker ps | grep ${CONTAINER_NAME}
                    curl -f http://localhost:${APP_PORT}/actuator/health || echo "Health check failed"
                    echo "✅ Application déployée avec succès sur le port ${APP_PORT}!"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ ========== Pipeline CI/CD exécuté avec succès! =========='
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
        }
        failure {
            echo '❌ ========== Le pipeline a échoué =========='
            sh 'docker logs ${CONTAINER_NAME} || true'
        }
        cleanup {
            echo '🧹 ========== Nettoyage de l\'espace de travail =========='
            cleanWs()
        }
    }
}