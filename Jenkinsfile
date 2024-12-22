pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'  // URL de SonarQube
        SONAR_LOGIN = credentials('sonarqube-token')  // Token sécurisé pour SonarQube
        MAVEN_REPO_URL = 'https://mymavenrepo.com/repo/XXXX/'
        MAVEN_REPO_USERNAME = credentials('maven-username') // Crédential sécurisé
        MAVEN_REPO_PASSWORD = credentials('maven-password') // Crédential sécurisé
        SLACK_WEBHOOK = credentials('slack-webhook') // URL webhook pour Slack
        EMAIL_RECIPIENTS = 'team@example.com'
    }

    stages {
        // === PHASE 1: TEST ===
        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh './gradlew test'  // Lancement des tests unitaires
            }
            post {
                always {
                    junit '**/build/test-results/test/*.xml' // Archivage des résultats
                }
                failure {
                    script {
                        notifyFailure("Unit tests failed")
                    }
                }
            }
        }

        stage('Generate Cucumber Reports') {
            steps {
                echo 'Generating Cucumber reports...'
                sh './gradlew test'  // Re-génération avec plugin Cucumber
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'build/reports/cucumber-html-reports',
                        reportFiles: 'overview-features.html',
                        reportName: 'Cucumber Test Report'
                    ])
                }
            }
        }

        // === PHASE 2: CODE ANALYSIS ===
        stage('Code Analysis with SonarQube') {
            steps {
                echo 'Running SonarQube analysis...'
                sh './gradlew sonarqube -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_LOGIN}'
            }
            post {
                failure {
                    script {
                        notifyFailure("SonarQube analysis failed")
                    }
                }
            }
        }

        // === PHASE 3: CODE QUALITY CHECK ===
        stage('Check Quality Gates') {
            steps {
                echo 'Checking SonarQube Quality Gates...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true // Arrête si Quality Gate échoue
                }
            }
        }

        // === PHASE 4: BUILD ===
        stage('Build Jar and Documentation') {
            steps {
                echo 'Building Jar file and generating documentation...'
                sh './gradlew jar javadoc'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                    archiveArtifacts artifacts: 'build/docs/javadoc/**', fingerprint: true
                }
            }
        }

        // === PHASE 5: DEPLOY ===
        stage('Deploy to Maven Repo') {
            steps {
                echo 'Deploying to Maven repository...'
                sh """
                ./gradlew publish \
                -Dmaven.repo.url=${MAVEN_REPO_URL} \
                -Dmaven.repo.username=${MAVEN_REPO_USERNAME} \
                -Dmaven.repo.password=${MAVEN_REPO_PASSWORD}
                """
            }
            post {
                success {
                    script {
                        notifySuccess("Deployment to Maven Repo successful!")
                    }
                }
                failure {
                    script {
                        notifyFailure("Deployment failed!")
                    }
                }
            }
        }

        // === PHASE 6: NOTIFICATION ===
        stage('Notify Team') {
            steps {
                script {
                    notifySuccess("Pipeline executed successfully!")
                }
            }
        }
    }

    post {
        failure {
            script {
                notifyFailure("Pipeline failed!")
            }
        }
    }
}

def notifySuccess(String message) {
   // slackSend(channel: '#team-channel', color: 'good', message: message)
    mail(to: "${EMAIL_RECIPIENTS}", subject: 'Pipeline Success', body: message)
}

def notifyFailure(String message) {
    /*slackSend(channel: '#team-channel', color: 'danger', message: message)*/
    mail(to: "${EMAIL_RECIPIENTS}", subject: 'Pipeline Failed', body: message)
}
