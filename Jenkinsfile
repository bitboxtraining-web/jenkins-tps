pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    try {
                        echo "Compilation en cours..."
                        sh 'exit 1'  // Simulation d'une erreur
                    } catch (Exception e) {
                        echo "Erreur d�tect�e dans le build !"
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }
    post {
        failure {
            echo "Le pipeline a �chou�, envoi d'une notification..."
            sh 'echo "Erreur d�tect�e" > erreur.log'
            archiveArtifacts artifacts: 'erreur.log', fingerprint: true
        }
        success {
            echo "Pipeline ex�cut� avec succ�s !"
        }
    }
}