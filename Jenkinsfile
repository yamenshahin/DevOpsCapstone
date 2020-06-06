pipeline {
  agent any

  environment {
    registry = "yamenshahin/docker-test"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  stages {
    stage('Linting Docker') {
      steps {
        script {
          sh 'hadolint Dockerfile'
        }
      }
    }
    stage('Linting React app') {
      steps {
        script {
          sh '''#!/bin/bash
          # Run Eslint
          npm run lint
          '''
        }
      }
    }
    stage('Building image') {
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Deploy Image') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', registryCredential) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps {
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
  }
}