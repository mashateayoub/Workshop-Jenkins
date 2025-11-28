# Workshop : Mettre en Place une CI/CD avec Jenkins pour un Projet MERN

## Objectifs du Workshop

À la fin de cet atelier, vous serez capable de :

-   Comprendre les concepts fondamentaux de l'Intégration Continue (CI) et du Déploiement Continu (CD).
-   Installer et configurer un serveur Jenkins à l'aide de Docker.
-   Créer un pipeline Jenkins complet pour une application MERN (MongoDB, Express, React, Node.js).
-   Automatiser les étapes de build, de test et de déploiement (simulé) de vos applications frontend et backend.
-   Utiliser un `Jenkinsfile` pour définir votre pipeline en tant que code (Pipeline-as-Code).

## Prérequis

-   **Git et GitHub** : Connaissances de base de `git` et un compte GitHub.
-   **Docker** : Docker et Docker Compose doivent être installés sur votre machine.
-   **Projet MERN** : Un projet MERN simple, séparé en deux dossiers (`/backend` et `/frontend`), avec des `Dockerfile` à la racine de chacun. Chaque projet doit contenir des scripts `npm install` et `npm test`.

---

## Partie 1 : Introduction à la CI/CD et à Jenkins

### Qu'est-ce que l'Intégration Continue (CI) ?

L'Intégration Continue est une pratique de développement où les développeurs intègrent leur code dans un référentiel partagé plusieurs fois par jour. Chaque intégration est ensuite automatiquement vérifiée par un **build** et des **tests automatisés**.

**Objectif** : Détecter les problèmes d'intégration le plus tôt possible pour réduire les bugs et améliorer la qualité du logiciel.

### Qu'est-ce que le Déploiement Continu (CD) ?

Le Déploiement Continu va un cran plus loin. Si l'étape de CI est réussie (build et tests OK), le code est automatiquement déployé en production (ou dans un environnement de pré-production).

**Objectif** : Livrer de nouvelles fonctionnalités aux utilisateurs de manière rapide, efficace et fiable.

### Pourquoi Jenkins ?

Jenkins est un serveur d'automatisation open-source qui est devenu un standard de l'industrie pour la CI/CD.

-   **Extensible** : Un écosystème de plus de 1800 plugins pour s'intégrer à presque tous les outils (Git, Docker, AWS, etc.).
-   **Flexible** : Permet de créer des pipelines simples comme des workflows d'orchestration très complexes.
-   **Pipeline-as-Code** : La possibilité de définir son pipeline dans un fichier (`Jenkinsfile`) qui est versionné avec le code de l'application.

---

## Partie 2 : Installation et Configuration de Jenkins

### Lancer Jenkins avec Docker

La manière la plus simple et la plus isolée de lancer Jenkins est via Docker. Ouvrez un terminal et exécutez la commande suivante :

```bash
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins \
  jenkins/jenkins:lts-jdk11
```
- `-d` : Lance le conteneur en arrière-plan.
- `-p 8080:8080` : Expose l'interface web de Jenkins.
- `-v jenkins_home:/var/jenkins_home` : Crée un volume pour persister les données de Jenkins.

### Première Connexion

1.  **Récupérer le mot de passe** : Jenkins va générer un mot de passe initial. Pour le récupérer, exécutez :
    ```bash
    docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
    ```
2.  **Accéder à Jenkins** : Ouvrez votre navigateur à l'adresse `http://localhost:8080`.
3.  **Configuration Initiale** :
    -   Collez le mot de passe récupéré.
    -   Cliquez sur **"Install suggested plugins"**. Jenkins installera les plugins les plus courants.
    -   Créez votre premier **utilisateur administrateur**.

### Installer les Plugins pour MERN

Une fois connecté, allez dans **Manage Jenkins > Manage Plugins > Available**. Recherchez et installez les plugins suivants :
-   `NodeJS` (pour utiliser `npm`)
-   `Docker Pipeline` (pour interagir avec Docker depuis le pipeline)

---

## Partie 3 : Création de notre Premier Pipeline

Nous allons créer un projet "Pipeline" qui lira sa configuration depuis un `Jenkinsfile` dans notre dépôt Git.

1.  **Créer un nouvel item** : Sur la page d'accueil de Jenkins, cliquez sur **"New Item"**.
2.  **Nommer le projet** : Donnez-lui un nom (ex: `MERN-app-pipeline`).
3.  **Choisir le type "Pipeline"** et cliquez sur OK.
4.  **Configurer le Pipeline** :
    -   Dans l'onglet **"General"**, vous pouvez cocher "GitHub project" et y mettre l'URL de votre dépôt.
    -   Dans la section **"Pipeline"**, changez la **"Definition"** pour **"Pipeline script from SCM"**.
    -   **SCM** : Choisissez **"Git"**.
    -   **Repository URL** : Mettez l'URL de votre dépôt GitHub (ex: `https://github.com/votre-nom/votre-projet-mern.git`).
    -   **Branch Specifier** : Laissez `*/main` ou changez pour votre branche principale.
    -   **Script Path** : Laissez `Jenkinsfile`. C'est le nom du fichier que Jenkins cherchera à la racine de votre projet.
5.  **Sauvegardez**.

---

## Partie 4 : Écriture du `Jenkinsfile`

Le `Jenkinsfile` est le cœur de notre CI/CD. Il décrit toutes les étapes que Jenkins doit exécuter. Nous utiliserons la syntaxe **déclarative**, qui est plus moderne et plus lisible.

Créez un fichier nommé `Jenkinsfile` à la racine de votre projet MERN.

### Structure de Base

```groovy
pipeline {
    agent any // Définit où le pipeline s'exécutera. 'any' signifie sur n'importe quel agent disponible.

    tools {
        nodejs 'NodeJS-16' // Nom de l'installation NodeJS configurée dans Jenkins
    }

    stages {
        // Nos étapes viendront ici
    }
}
```

### 📖 Explication de la Syntaxe du Jenkinsfile

Avant d'aller plus loin, clarifions chaque élément de la syntaxe :

#### `pipeline { }`
- **Définition** : Le bloc racine qui encapsule toute la définition du pipeline.
- **Obligatoire** : Oui, c'est le conteneur principal de votre pipeline déclaratif.
- **Exemple** : Tout votre code Jenkins doit être à l'intérieur de ce bloc.

#### `agent`
- **Définition** : Spécifie **où** et **comment** le pipeline (ou une étape spécifique) sera exécuté.
- **Options courantes** :
  - `agent any` : Exécute sur n'importe quel agent disponible.
  - `agent none` : Aucun agent global ; chaque stage doit définir son propre agent.
  - `agent { docker { image 'node:16' } }` : Exécute dans un conteneur Docker spécifique.
  - `agent { label 'linux' }` : Exécute sur un agent avec le label "linux".
- **Utilisation** : Peut être défini au niveau du pipeline (global) ou au niveau de chaque `stage`.

#### `tools { }`
- **Définition** : Déclare les outils (comme Node.js, Maven, JDK) que Jenkins doit rendre disponibles dans l'environnement.
- **Prérequis** : Les outils doivent être configurés dans **Manage Jenkins > Global Tool Configuration**.
- **Exemple** :
  ```groovy
  tools {
      nodejs 'NodeJS-16'  // Nom configuré dans Jenkins
      maven 'Maven-3.8'
  }
  ```
- **Note** : Le nom (ex: `'NodeJS-16'`) doit correspondre **exactement** au nom que vous avez donné dans la configuration globale.

#### `stages { }`
- **Définition** : Conteneur qui regroupe toutes les **étapes** (stages) de votre pipeline.
- **Obligatoire** : Oui, un pipeline doit contenir au moins un `stage`.
- **Structure** : Contient un ou plusieurs blocs `stage`.

#### `stage('Nom de l'étape') { }`
- **Définition** : Représente une **phase logique** de votre pipeline (ex: Build, Test, Deploy).
- **Nom** : Le nom entre guillemets apparaîtra dans l'interface Jenkins pour suivre la progression.
- **Exemple** :
  ```groovy
  stage('Build') {
      steps {
          // Actions à exécuter
      }
  }
  ```

#### `steps { }`
- **Définition** : Contient les **actions concrètes** à exécuter dans un stage.
- **Commandes courantes** :
  - `sh 'commande'` : Exécute une commande shell (Linux/Mac).
  - `bat 'commande'` : Exécute une commande batch (Windows).
  - `echo 'message'` : Affiche un message dans les logs.
  - `checkout scm` : Récupère le code source depuis le SCM (Git).
  - `dir('chemin') { }` : Change de répertoire pour les commandes à l'intérieur.

#### `parallel { }`
- **Définition** : Permet d'exécuter plusieurs stages **en parallèle** pour gagner du temps.
- **Exemple** :
  ```groovy
  stage('Tests Parallèles') {
      parallel {
          stage('Test Backend') {
              steps { sh 'npm test' }
          }
          stage('Test Frontend') {
              steps { sh 'npm test' }
          }
      }
  }
  ```

#### `script { }`
- **Définition** : Permet d'utiliser du code Groovy **scriptif** (plus flexible) à l'intérieur d'un pipeline déclaratif.
- **Utilisation** : Pour des logiques conditionnelles complexes, des boucles, ou l'utilisation de variables.
- **Exemple** :
  ```groovy
  steps {
      script {
          def version = sh(script: 'git describe --tags', returnStdout: true).trim()
          echo "Version: ${version}"
      }
  }
  ```

#### `post { }`
- **Définition** : Définit des actions à exécuter **après** l'exécution du pipeline, selon le résultat.
- **Conditions disponibles** :
  - `always` : Toujours exécuté, quel que soit le résultat.
  - `success` : Exécuté uniquement si le pipeline réussit.
  - `failure` : Exécuté uniquement si le pipeline échoue.
  - `unstable` : Exécuté si le build est instable (ex: tests échoués mais build OK).
  - `changed` : Exécuté si le résultat est différent du build précédent.
- **Exemple** :
  ```groovy
  post {
      always {
          echo 'Nettoyage...'
      }
      success {
          echo 'Build réussi !'
      }
      failure {
          echo 'Build échoué !'
      }
  }
  ```

#### `environment { }`
- **Définition** : Définit des **variables d'environnement** disponibles dans tout le pipeline ou dans un stage spécifique.
- **Exemple** :
  ```groovy
  environment {
      NODE_ENV = 'production'
      API_URL = 'https://api.example.com'
  }
  ```

#### `dir('chemin') { }`
- **Définition** : Change le répertoire de travail pour les commandes à l'intérieur du bloc.
- **Exemple** :
  ```groovy
  steps {
      dir('backend') {
          sh 'npm install'  // Exécuté dans le dossier backend
      }
  }
  ```

#### `withCredentials([ ]) { }`
- **Définition** : Injecte des **credentials** (mots de passe, tokens) de manière sécurisée dans l'environnement.
- **Exemple** :
  ```groovy
  withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
      sh "docker login -u ${USER} -p ${PASS}"
  }
  ```

---

> **Note** : Avant de continuer, allez dans **Manage Jenkins > Global Tool Configuration**, trouvez la section **NodeJS** et ajoutez une installation. Donnez-lui le nom `NodeJS-16` et choisissez une version récente.

### Pipeline pour un Projet MERN (Approche Monorepo)

Voici un exemple complet pour un projet où `/frontend` et `/backend` sont dans le même dépôt.

```groovy
pipeline {
    agent any

    tools {
        nodejs 'NodeJS-16' // Assurez-vous que ce nom correspond à votre configuration
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupère le code source de la branche qui a déclenché le build
                checkout scm
            }
        }

        stage('Build & Test Parallel') {
            parallel {
                stage('Backend') {
                    steps {
                        dir('backend') {
                            echo '--- Building & Testing Backend ---'
                            sh 'npm install'
                            sh 'npm test'
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        dir('frontend') {
                            echo '--- Building & Testing Frontend ---'
                            sh 'npm install'
                            sh 'npm test'
                            sh 'npm run build'
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Backend Image') {
                    steps {
                        dir('backend') {
                            echo '--- Building Backend Docker Image ---'
                            script {
                                // 'myapp-backend' est le nom de l'image
                                def backendImage = docker.build('myapp-backend:latest')
                            }
                        }
                    }
                }
                stage('Build Frontend Image') {
                    steps {
                        dir('frontend') {
                            echo '--- Building Frontend Docker Image ---'
                            script {
                                def frontendImage = docker.build('myapp-frontend:latest')
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy (Simulation)') {
            steps {
                echo '--- Deploying services with Docker Compose ---'
                // Dans un vrai scénario, vous vous connecteriez à un serveur distant
                // et lanceriez docker-compose up. Ici, on simule.
                // Assurez-vous d'avoir un docker-compose.yml à la racine.
                sh 'docker-compose up -d'
                
                // Attendre quelques secondes pour que les services démarrent
                sleep 10
                
                echo '--- Deployment Simulation Complete ---'
            }
        }
    }

    post {
        always {
            // Nettoyer l'environnement après le build
            echo '--- Cleaning up ---'
            sh 'docker-compose down'
        }
        success {
            echo 'Pipeline Succeeded!'
            // Envoyer une notification de succès (ex: Slack, email)
        }
        failure {
            echo 'Pipeline Failed!'
            // Envoyer une notification d'échec
        }
    }
}
```

---

## Partie 5 : Bonnes Pratiques et Concepts Intermédiaires

### Webhooks GitHub

Pour que Jenkins lance un build automatiquement à chaque `push` sur votre branche, vous devez configurer un webhook.

1.  **Dans GitHub** : Allez dans les **Settings** de votre dépôt > **Webhooks** > **Add webhook**.
2.  **Payload URL** : Entrez `http://<VOTRE_IP_PUBLIQUE>:8080/github-webhook/`. (Pour un test en local, des outils comme `ngrok` sont nécessaires pour exposer votre `localhost`).
3.  **Content type** : `application/json`.
4.  **Laissez les autres options par défaut** et cliquez sur **Add webhook**.

### Gérer les Secrets (Credentials)

Ne mettez jamais de mots de passe, de tokens ou de clés API en clair dans votre `Jenkinsfile`. Utilisez le **Credentials Manager** de Jenkins.

1.  Allez dans **Manage Jenkins > Manage Credentials**.
2.  Cliquez sur **(global)** puis **Add Credentials**.
3.  Choisissez le type de secret (ex: "Secret text" pour un token, "Username with password" pour Docker Hub).
4.  Donnez-lui un **ID** (ex: `DOCKER_HUB_CREDS`).

Ensuite, dans votre `Jenkinsfile`, vous pouvez l'utiliser de manière sécurisée :

```groovy
stage('Push to Docker Hub') {
    steps {
        script {
            // 'DOCKER_HUB_CREDS' est l'ID que vous avez défini
            withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDS', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                sh "docker login -u ${USER} -p ${PASS}"
                // docker.build(...).push()
            }
        }
    }
}
```

## Conclusion et Prochaines Étapes

Félicitations ! Vous avez mis en place un pipeline CI/CD fonctionnel pour une application MERN avec Jenkins. Vous avez automatisé le processus de test, de build et de déploiement (simulé), tout en suivant les meilleures pratiques comme le Pipeline-as-Code.

**Pour aller plus loin :**
-   Mettre en place un vrai déploiement sur un serveur distant (via SSH).
-   Utiliser des registres d'images Docker comme Docker Hub ou AWS ECR.
-   Explorer des stratégies de déploiement plus avancées comme le Blue/Green.
-   Intégrer des outils d'analyse de code statique (SonarQube).
-   Gérer différents environnements (développement, pré-production, production).