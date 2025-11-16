pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    stages {

        stage('Build Maven') {
            steps {
                dir('demo-backend') {
                    sh 'mvn -B -DskipTests package'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'demo-backend/target/*.jar', fingerprint: true
            }
        }

    }
    
}
