pipeline {
    agent {
        docker {
            image 'maven:3.9.12-eclipse-temurin-21-alpine'
            args '-v $HOME/.m2:/root/.m2'
        }
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
            echo 'Pipeline réussi ! ✅'
            // Tu peux ajouter une notification ici
        }
        failure {
            echo 'Pipeline échoué ! ❌'
        }
    }
}
