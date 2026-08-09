@Library('jenkins-shared-library') _

pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = credentials('aws-account-id')
        AWS_REGION     = 'ap-south-1'
    }

    stages {
        stage('Checkout') {
            steps {
                gitCheckout(env.GIT_URL)
            }
        }
        stage('Build') {
            steps {
                mavenBuild()
            }
        }
        stage('Docker Build') {
            steps {
                dockerBuildImage('maven-app', "v${BUILD_NUMBER}")
            }
        }
        stage('Docker Push') {
            steps {
                dockerPushImage("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/maven-app", "v${BUILD_NUMBER}")
            }
        }
    }
}