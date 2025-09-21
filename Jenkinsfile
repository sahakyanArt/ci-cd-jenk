@Library('jenkins-shared-lib@main') _

pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Lint Dockerfile') {
            steps {
                dockerLint()
            }
        }

        stage('Build Docker Image') {
            steps {
                dockerBuildImage(branch: env.BRANCH_NAME)
            }
        }

        stage('Scan Docker Image') {
            steps {
                dockerScanImage(branch: env.BRANCH_NAME)
            }
        }

        stage('Deploy') {
            steps {
                deployContainer(branch: env.BRANCH_NAME)
            }
        }
    }
}
