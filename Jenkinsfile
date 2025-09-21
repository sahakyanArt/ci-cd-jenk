@Library('jenkins-shared-lib@main') _
def docker = dockerUtils()

pipeline {
    agent any
    stages {
        stage('Lint') { steps { script { docker.lint() } } }
        stage('Build Image') { steps { script { docker.buildImage(env.BRANCH_NAME) } } }
        stage('Scan') { steps { script { docker.scanImage(env.BRANCH_NAME) } } }
        stage('Deploy') { steps { script { docker.deploy(env.BRANCH_NAME) } } }
    }
}

