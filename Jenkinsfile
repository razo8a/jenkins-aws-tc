pipeline {
    agent any
    triggers {
        githubPullRequest()
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Unit Tests') {
            steps {
                sh './run-unit-tests.sh' // Replace with your unit test command
            }
        }
    }
    post {
        success {
            githubNotify context: 'Unit Tests', status: 'SUCCESS', description: 'All tests passed.'
        }
        failure {
            githubNotify context: 'Unit Tests', status: 'FAILURE', description: 'Tests failed.'
        }
    }
}
