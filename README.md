# TP : Pipeline CI/CD avec Jenkins Multibranch et GitHub

## 📋 Description du projet
Ce projet démontre la mise en place d'un pipeline CI/CD (Intégration Continue / Déploiement Continu) automatisé utilisant Jenkins Multibranch Pipeline et GitHub. À chaque push sur le dépôt GitHub, un pipeline Jenkins se déclenche automatiquement pour construire, tester et valider le projet.

## 🎯 Objectifs réalisés
✅ Configuration d'un pipeline Jenkins Multibranch

✅ Intégration avec GitHub via webhooks

✅ Exposition de Jenkins local via ngrok

✅ Pipeline exécuté dans un conteneur Docker avec Maven

✅ Déclenchement automatique à chaque push GitHub

✅ Construction et tests automatisés d'une application Java

## 🏗️ Architecture
```
GitHub (Push) → Webhook → ngrok → Jenkins Local → Docker Container → Build Maven → Tests
```

## 📁 Structure du projet
```
MultiBranchPipeLine-Elkhattabimohamedamine/
├── Jenkinsfile              # Définition du pipeline CI/CD
├── pom.xml                  # Configuration Maven
├── src/                     # Code source de l'application
│   ├── main/java/com/example/App.java
│   └── test/java/
├── images/                  # Captures d'écran
├── target/                  # Artéfacts de build (générés)
└── README.md               # Ce fichier
```

## 🔧 Configuration requise
- Jenkins 2.264+ avec plugins :
  - Pipeline
  - GitHub Integration
  - Docker Pipeline
- Docker pour l'exécution des conteneurs
- Compte GitHub avec un dépôt
- ngrok pour l'exposition de Jenkins
- Java 21 et Maven 3.9+ (dans le conteneur)

## 🚀 Installation et configuration

### 1. Configuration Jenkins
```bash
# Installation des plugins requis
- GitHub Integration Plugin
- Pipeline
- Docker Pipeline

# Configuration de la sécurité
1. Manage Jenkins → Configure Global Security
2. Allow anonymous read access (pour les webhooks)
3. OU configurer CSRF avec exclusion /github-webhook/
```

### 2. Configuration ngrok
```bash
# Authentification
ngrok config add-authtoken <VOTRE_TOKEN>

# Exposition de Jenkins
ngrok http 8080
# URL générée : https://xxxx-xx-xx-xx-xx.ngrok-free.app
```

### 3. Configuration GitHub Webhook
- Allez dans : Settings → Webhooks → Add webhook
- Payload URL : `https://[URL_NGROK]/github-webhook/`
- Content type : application/json
- Événements : Just the push event
- Activez le webhook

### 4. Création du Multibranch Pipeline dans Jenkins
- New Item → Multibranch Pipeline
- Branch Sources → Git
- Entrez l'URL de votre dépôt GitHub
- Save → Jenkins scanne automatiquement les branches

## 📜 Jenkinsfile
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9.12-eclipse-temurin-21-alpine'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    
    stages {
        stage('Vérification (Checkout)') {
            steps {
                checkout scm
            }
        }
        
        stage('Construction (Build)') {
            steps {
                sh 'mvn -B -DskipTests clean package'
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
    }
}
```

## 📸 Captures d'écran

### 1. Pipeline Multibranch réussi
![Pipeline Success](images/multibranch%20pipeline%20success.png)
*Capture montrant le pipeline Jenkins avec toutes les étapes réussies*

### 2. Webhook GitHub fonctionnel
![Ngrok Delivery](images/ngrok%20successful%20selivery.png)
*Page GitHub Webhooks montrant les délivrances réussies via ngrok*

## 🔍 Tests effectués

### Test 1 : Déclenchement par push
- Modification du fichier README.md
- Commit et push :
```bash
git add README.md
git commit -m "Test déclenchement CI/CD"
git push origin main
```
- **Résultat** : Build automatiquement déclenché dans Jenkins

### Test 2 : Nouvelle branche
- Création d'une branche feature :
```bash
git checkout -b feature/test-ci
git push origin feature/test-ci
```
- **Résultat** : Jenkins détecte automatiquement la nouvelle branche et exécute le pipeline

### Test 3 : Échec de build simulé
- Introduction d'une erreur de compilation
- Push sur la branche
- **Résultat** : Jenkins montre clairement l'étape en échec avec les logs d'erreur

## ⚠️ Problèmes rencontrés et solutions

### Problème 1 : Erreur 403 Forbidden du webhook
**Symptôme** : GitHub reçoit une réponse 403 de Jenkins

**Solution** :
- Désactiver temporairement CSRF dans Jenkins
- OU autoriser l'accès anonyme en lecture
- OU configurer une exclusion CSRF pour `/github-webhook/`

### Problème 2 : Accès Docker refusé
**Symptôme** : `permission denied while trying to connect to Docker daemon`

**Solution** :
```bash
docker exec -u root -it jenkins bash
groupadd docker || true
usermod -aG docker jenkins
chmod 666 /var/run/docker.sock
docker restart jenkins
```

### Problème 3 : Ngrok URL qui change
**Symptôme** : L'URL ngrok change à chaque redémarrage

**Solution** : Pour un environnement stable, utiliser un compte ngrok payant avec sous-domaine personnalisé

## 📝 Conclusion
Ce TP démontre avec succès l'implémentation d'un pipeline CI/CD complet avec Jenkins Multibranch, intégration GitHub et exposition via ngrok. Le système répond automatiquement aux changements de code et exécute les tests dans un environnement containerisé.