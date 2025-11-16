pipeline {
    agent any

    stages {
        stage('Build Maven') {
            steps {
                sh 'mvn -B -DskipTests package'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}
