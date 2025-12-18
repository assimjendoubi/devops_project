pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }
    
    environment {
        SONAR_HOST_URL = 'http://192.168.33.10:9000'
    }
    
    stages {
        stage('📥 Checkout') {
            steps {
                echo '========== Récupération du code source depuis Git =========='
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
                echo '========== Analyse de la qualité du code avec SonarQube =========='
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey=tp-projet-2025 \
                            -Dsonar.projectName="TP Projet 2025" \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.token=${SONAR_TOKEN}
                        '''
                    }
                }
            }
        }
        
        stage('📦 Package') {
            steps {
                echo '========== Génération du fichier JAR =========='
                sh 'mvn package -DskipTests'
            }
        }
    }
    
    post {
        success {
            echo '✅ ========== Pipeline CI exécuté avec succès! =========='
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
        failure {
            echo '❌ ========== Le pipeline a échoué =========='
        }
        always {
            echo '🧹 ========== Nettoyage de l\'espace de travail =========='
            cleanWs()
        }
    }
}
