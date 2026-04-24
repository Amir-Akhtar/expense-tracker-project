environment {
    RENDER_API_KEY = credentials('render-api-key')

    RENDER_BACKEND_DEPLOY_HOOK = "https://api.render.com/deploy/srv-d7l2p2gsfn5c73cs9b0g?key=UaZzMxXX_YQ"
    RENDER_FRONTEND_DEPLOY_HOOK = "https://api.render.com/deploy/srv-d7l20tn7f7vs73ap1uvg?key=Jf9DQVp-fpU"
    
}
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
                    sh 'mvn clean compile'
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
        }/*
        stage('Sonar') {
            steps {
                withSonarQubeEnv('MySonarServer') {
                    dir('expense-tracker-service') {
                        sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=expense-tracker \
                        -Dsonar.host.url=http://192.168.57.14:9000 \
                        -Dsonar.login=sq-token
                        '''
                    }
                }
            }
        }*/
        stage ('Sonar Checks') {
            steps {
                withSonarQubeEnv('MySonarServer') {
                    dir('expense-tracker-service') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
            post {
                success {
                    script {
                        timeout(time: 2, unit: 'MINUTES') {
                            def qualityGate = waitForQualityGate()
                            if (qualityGate.status != 'OK') {
                                error "SonarQube Quality Gate failed: ${qualityGate.status}"
                            } else {
                                echo " SonarQube Analysis Passed."
                            }
                        }
                    }
                }
                failure {
                    echo "SonarQube analysis failed during execution."
                }
            }
        }
        stage('Deploy to Render') {
            steps {
                script {
                    echo "Deploying Backend..."
                    def backendResponse = httpRequest(
                        url: "${RENDER_BACKEND_DEPLOY_HOOK}",
                        httpMode: 'POST',
                        validResponseCodes: '200:299'
                    )
                    echo "Render Backend Deployment Response: ${backendResponse}"
        
                    echo "Deploying Frontend..."
                    def frontendResponse = httpRequest(
                        url: "${RENDER_FRONTEND_DEPLOY_HOOK}",
                        httpMode: 'POST',
                        validResponseCodes: '200:299'
                    )
                    echo "Render Frontend Deployment Response: ${frontendResponse}"
                }
            }
        }
    }

    post {
        success {
            echo 'CI Build Successful'
        }

        failure {
            echo 'CI Build Failed'
        }
    }
}
