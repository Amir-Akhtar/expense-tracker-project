pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    tools {
        maven 'mvn'
        nodejs 'node'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/Amir-Akhtar/expense-tracker-project.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('expense-tracker-service') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('expense-tracker-ui') {
                    sh 'npm install'
                    sh 'npm run build -- --configuration production'
                }
            }
        }

        stage('Test Backend') {
            steps {
                dir('expense-tracker-service') {
                    sh 'mvn test'
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI Build Successful'
        }

        failure {
            echo '❌ CI Build Failed'
        }
    }
}
