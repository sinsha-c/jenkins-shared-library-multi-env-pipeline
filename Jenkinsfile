@Library('jenkins-shared-library') _

pipeline {
    agent any

    environment {
        REPO_URL   = 'https://github.com/your-username/your-app.git'
        REPO_BRANCH = 'main'
        IMAGE_NAME = 'your-app'
    }

    stages {
        stage('Checkout') {
            steps {
                gitCheckout(env.REPO_URL, env.REPO_BRANCH)
            }
        }
        stage('Build') {
            steps {
                mavenBuild()
            }
        }
        stage('Docker Build') {
            steps {
                dockerBuildImage(env.IMAGE_NAME, "${env.BUILD_NUMBER}")
            }
        }
        stage('Docker Push') {
            steps {
                dockerPushImage(env.IMAGE_NAME, "${env.BUILD_NUMBER}")
            }
        }
    }
}