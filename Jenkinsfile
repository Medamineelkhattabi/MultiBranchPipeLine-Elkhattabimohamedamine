pipeline {
    agent {
        docker {
            image 'maven:3.9.12-eclipse-temurin-21-alpine'
            args '-v $HOME/.m2:/root/.m2 -v /tmp/maven-repo:/tmp/.m2'
        }
    }
    
    environment {
        MAVEN_OPTS = '-Dmaven.repo.local=/tmp/.m2'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn -B clean compile'
            }
        }
        
        stage('Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Archive') {
            steps {
                archiveArtifacts 'target/*.jar'
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline Maven RÉUSSI !'
            echo 'Projet compilé, testé et packagé avec succès.'
        }
        failure {
            echo '❌ Pipeline échoué !!'
        }
        always {
            echo '--- Fin de l\'exécution du pipeline ---'
        }
    }
}