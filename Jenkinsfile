@Library('jenkins-shared-library') _
 
pipeline {
    agent any
 
    environment {
        AWS_ACCOUNT_ID = credentials('aws-account-id')
        AWS_REGION     = 'ap-south-1'
    }
 
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['Dev', 'QA', 'Staging', 'Production'],
            description: 'Select the environment to deploy to'
        )
    }
 
    stages {
        stage('Show Parameters') {
            steps {
                echo "Selected Environment: ${params.ENVIRONMENT}"
            }
        }
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
                dockerBuild('maven-app', "${params.ENVIRONMENT.toLowerCase()}")
            }
        }
        stage('Docker Push') {
            steps {
                dockerPush('maven-app', "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/maven-app", "${params.ENVIRONMENT.toLowerCase()}")
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying maven-app to ${params.ENVIRONMENT} environment..."
            }
        }
    }
}