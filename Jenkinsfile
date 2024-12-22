pipeline {
    agent any

    environment {
        SONAR_SERVER = 'sonar' // Assurez-vous que ce nom est correct
        SLACK_CHANNEL = '#your-channel'
        EMAIL_RECIPIENTS = 'developer@example.com'
    }

    stages {
        stage('Test') {
            steps {
                bat './gradlew test'
                junit '**/build/test-results/test/*.xml'
                cucumber buildStatus: 'UNSTABLE', reportTitle: 'Cucumber Report', fileIncludePattern: 'build/cucumber-reports/*.json'
            }
        }

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv(SONAR_SERVER) {
                    bat './gradlew sonarqube'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Build') {
            steps {
                bat './gradlew build'
                archiveArtifacts artifacts: 'build/libs/*.jar', allowEmptyArchive: true
                archiveArtifacts artifacts: '**/build/docs/javadoc/**/*', allowEmptyArchive: true
            }
            post {
                success {
                    //slackSend(channel: SLACK_CHANNEL, color: 'good', message: 'Build Succeeded!')
                    emailext(
                        subject: 'Build Succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                        body: 'The build succeeded. Check the details at ${env.BUILD_URL}.',
                        to: EMAIL_RECIPIENTS
                    )
                }
                failure {
                   // slackSend(channel: SLACK_CHANNEL, color: 'danger', message: 'Build Failed!')
                    emailext(
                        subject: 'Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                        body: 'The build failed. Check the details at ${env.BUILD_URL}.',
                        to: EMAIL_RECIPIENTS
                    )
                }
            }
        }

        stage('Deploy') {
            steps {
                bat './gradlew publish'
            }
        }
    }
}
