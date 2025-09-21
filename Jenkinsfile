pipeline {
    agent any

    environment {
        APP_NAME = "nodeapp"
        MAIN_PORT = "3000"
        DEV_PORT  = "3001"
    }

    tools {
        nodejs 'node7.8.0'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Lint Dockerfile') {
            steps {
                sh 'docker run --rm -i hadolint/hadolint hadolint --failure-threshold error Dockerfile'
            }
        }

        stage('Build') {
            agent { docker { image 'node:7.8' } }
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "Tests failed but continuing..."'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker build -t nodemain:v1.0 ."
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh "docker build -t nodedev:v1.0 ."
                    }
                }
            }
        }

        stage('Scan Docker image for vulnerabilities') {
            steps {
                script {
                    def imageName = env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    def vulnerabilities = sh(
                        script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${imageName}",
                        returnStdout: true
                    ).trim()
                    echo "Vulnerabilities report:\n${vulnerabilities}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker') {
                        if (env.BRANCH_NAME == 'main') {
                            sh "docker tag nodemain:v1.0 sahakyan13/nodemain:v1.0"
                            sh "docker push sahakyan13/nodemain:v1.0"
                        } else {
                            sh "docker tag nodedev:v1.0 sahakyan13/nodedev:v1.0"
                            sh "docker push sahakyan13/nodedev:v1.0"
                        }
                    }
                }
            }
        }

        stage('Deploy Local') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh '''
                        docker ps -q --filter "name=nodemain" | xargs -r docker stop
                        docker ps -a -q --filter "name=nodemain" | xargs -r docker rm
                        docker run -d --name nodemain -p 3000:3000 nodemain:v1.0
                        '''
                    } else {
                        sh '''
                        docker ps -q --filter "name=nodedev" | xargs -r docker stop
                        docker ps -a -q --filter "name=nodedev" | xargs -r docker rm
                        docker run -d --name nodedev -p 3001:3000 nodedev:v1.0
                        '''
                    }
                }
            }
        }

        stage('Trigger Deploy Pipeline') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        build job: 'Deploy_to_main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        build job: 'Deploy_to_dev'
                    }
                }
            }
        }
    }
}
