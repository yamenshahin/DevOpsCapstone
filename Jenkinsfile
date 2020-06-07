pipeline {
  agent any

  environment {
    registry = "yamenshahin/docker-test"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  stages {
    stage('Linting Index/HTML') {
      steps {
        script {
          sh 'tidy -q -e public/*.html'
        }
      }
    }
    stage('Linting Docker') {
      steps {
        script {
          sh 'hadolint Dockerfile'
        }
      }
    }
    stage('Building Image') {
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
    stage('Remove Unused Docker Image') {
      steps {
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
    stage('Set Current kubectl Context to The Cluster') {
      steps {
				withAWS(region:'us-east-1', credentials:'aws_credentials') {
					sh '''
						kubectl config use-context arn:aws:eks:us-east-1:655649737419:cluster/projectCapstone
					'''
				}
			}
    }
    stage('Create Blue Controller') {
      steps {
				withAWS(region:'us-east-1', credentials:'aws_credentials') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
    }
    stage('Create Grean Controller') {
      steps {
				withAWS(region:'us-east-1', credentials:'aws_credentials') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
    }
    stage('Create Service To Redirect to Blue') {
      steps {
				withAWS(region:'us-east-1', credentials:'aws_credentials') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
    }
    stage('Route to Green?') {
      steps {
        input "Do you want to redirect traffic to green?"
      }
    }
    stage('Create Service To Redirect to Green') {
      steps {
				withAWS(region:'us-east-1', credentials:'aws_credentials') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
    }
  }
}