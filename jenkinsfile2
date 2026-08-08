@Library('jenkins-shared-library') _

pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/your-username/your-app.git', description: 'Git repository to build')
        string(name: 'REPO_BRANCH', defaultValue: 'main', description: 'Branch to checkout')
        string(name: 'IMAGE_NAME', defaultValue: 'your-app', description: 'Docker image name')
    }

    stages {
        stage('Checkout') {
            steps {
                gitCheckout(params.REPO_URL, params.REPO_BRANCH)
            }
        }
        stage('Build') {
            steps {
                mavenBuild()
            }
        }
        stage('Docker Build') {
            steps {
                dockerBuildImage(params.IMAGE_NAME, "${env.BUILD_NUMBER}")
            }
        }
        stage('Docker Push') {
            steps {
                dockerPushImage(params.IMAGE_NAME, "${env.BUILD_NUMBER}")
            }
        }
    }
}