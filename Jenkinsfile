pipeline {
    agent any

    stages {
        // Stage for running tests
        stage('Test') {
            steps {
                // Run tests using Gradle
                bat './gradlew test'
                

                // Publish JUnit test results to Jenkins
                junit 'build/test-results/test/*.xml'

                // Publish Cucumber test results
                cucumber buildStatus: 'UNSTABLE',
                         reportTitle: 'My report',
                         fileIncludePattern: 'target/report.json'
            }
        }

        // Stage for code analysis using SonarQube
        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    // Run SonarQube analysis
                    bat './gradlew sonar'
                }
            }
        }

        // Stage for checking the quality gate results from SonarQube
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        // Stage for building the project
        stage('Build') {
            steps {
                // Run the build command
                bat './gradlew build'

                // Archive the JAR files generated during the build
                archiveArtifacts artifacts: 'build/libs/*.jar', allowEmptyArchive: true

                // Archive Javadoc files if they are generated
                archiveArtifacts artifacts: '**/build/docs/javadoc/**/*', allowEmptyArchive: true

                // Notify external service (Slack, etc.) about the build status
                notifyEvents message: '<h1>Building...</h1>', token: '4iwq3njk9vw0ui7irxh0yiqhed0rf2qb'
            }

            post {
                // Notifications upon successful build
                success {
                    notifyEvents message: '<h1>succeeded and built...</h1>', token: '4lwq3njk9vw0ui7irxh0yiqhed0rf2qb'
                }

                // Notifications if the build fails
                failure {
                    notifyEvents message: '<h1>failed to build...</h1>', token: '4iwq3njk9vw0ui7irxh0yiqhed0rf2qb'
                }
            }
        }

        // Stage for deploying the project
        stage('Deploy') {
            steps {
                // Run the publish command (e.g., to push the build artifacts to a remote repository)
                bat './gradlew publish'
            }
        }
    }
}
