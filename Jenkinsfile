pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }
    
    stages {
        stage('📥 Checkout') {
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
    }
    
    post {
        success {
            echo '✅ ========== Pipeline CI exécuté avec succès! =========='
            // Archiver les artefacts AVANT le nettoyage
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
        }
        failure {
            echo '❌ ========== Le pipeline a échoué =========='
        }
        cleanup {
            // Nettoyer APRÈS l'archivage
            echo '🧹 ========== Nettoyage de l\'espace de travail =========='
            cleanWs()
        }
    }
}
