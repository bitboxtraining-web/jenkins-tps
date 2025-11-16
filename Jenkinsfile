pipeline {

  agent any

  tools {
      maven 'maven3'
      jdk 'jdk17'
  }

  environment {
      REGISTRY = "vps-36f602ea.vps.ovh.net:5000"
      IMAGE_NAME = "backend-app"
      SONAR_HOST_URL = "http://vps-36f602ea.vps.ovh.net:9000"
      SONAR_TOKEN = credentials('sonar-token')
  }

  stages {

    /* ===========================
       CHECKOUT
       =========================== */
    stage('Checkout') {
      steps {
        checkout scm
        echo "Branche détectée : ${env.BRANCH_NAME}"
      }
    }

    /* ===========================
       BUILD MAVEN
       =========================== */
    stage('Build') {
      steps {
        dir("demo-backend") {
          sh "mvn -B clean install -DskipTests"
        }
      }
    }

    /* ===========================
       TESTS UNITAIRES
       =========================== */
    stage('Tests') {
      when {
        expression { env.BRANCH_NAME =~ /feature\/.*|develop|release\/.*|hotfix\/.*|main/ }
      }
      steps {
        dir("demo-backend") {
          sh "mvn test"
        }
      }
    }

    /* ===========================
       ANALYSE SONARQUBE
       =========================== */
    stage('SonarQube Analysis') {
      when {
        expression { env.BRANCH_NAME =~ /develop|release\/.*|hotfix\/.*|main/ }
      }
      steps {
        dir("demo-backend") {
          withSonarQubeEnv('sonar') {
            sh """
              mvn sonar:sonar \
              -Dsonar.projectKey=demo-backend \
              -Dsonar.host.url=${SONAR_HOST_URL} \
              -Dsonar.login=${SONAR_TOKEN}
            """
          }
        }
      }
    }

    /* ===========================
       DOCKER BUILD
       =========================== */
    stage('Docker Build') {
      when {
        expression { env.BRANCH_NAME =~ /release\/.*|hotfix\/.*|main/ }
      }
      steps {
        sh """
          docker build -t ${REGISTRY}/${IMAGE_NAME}:${BRANCH_NAME} .
        """
      }
    }

    /* ===========================
       DOCKER PUSH
       =========================== */
    stage('Docker Push') {
      when {
        expression { env.BRANCH_NAME =~ /hotfix\/.*|main/ }
      }
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'docker-registry',
            usernameVariable: 'REG_USER',
            passwordVariable: 'REG_PASS'
          )
        ]) {
          sh """
            echo "$REG_PASS" | docker login ${REGISTRY} -u "$REG_USER" --password-stdin
            docker push ${REGISTRY}/${IMAGE_NAME}:${BRANCH_NAME}
          """
        }
      }
    }

    /* ===========================
       DEPLOY PROD (MAIN)
       =========================== */
    stage('Deploy to Production') {
      when {
        branch "main"
      }
      steps {
        echo "🚀 Déploiement en production…"
        sh """
          docker pull ${REGISTRY}/${IMAGE_NAME}:main
          docker stop backend || true
          docker rm backend || true
          docker run -d --name backend -p 8080:8080 ${REGISTRY}/${IMAGE_NAME}:main
        """
      }
    }

  } // stages
} // pipeline
