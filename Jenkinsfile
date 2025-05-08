pipeline {
    agent any
    environment {
        PROJECT_ID = 'applied-arcanum-459112-v3'
        CLUSTER_NAME = 'wimb-kube'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = '9375fded-5f75-4b71-9376-a74792a84ba4'
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
	stage("Prepare env") {
            steps {
                script {
                    sh "cp /var/jenkins_home/.env ./code/.env"
                }
            }
        }
        stage('Build image') {
            steps {
                script {
                    myapp = docker.build("crolvlee/wimb-2025:${env.BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'crolvlee') {
                        myapp.push("latest")
                        myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }
        stage('Deploy to GKE') {
			when {
				branch 'main'
			}
            steps{
                sh "sed -i 's/wimb-2025:latest/wimb-2025:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', 
                    projectId: env.PROJECT_ID, 
                    clusterName: env.CLUSTER_NAME, 
                    location: env.LOCATION, 
                    manifestPattern: 'deployment.yaml', 
                    credentialsId: env.CREDENTIALS_ID, 
                    verifyDeployments: true])
            }
        }
    }
}
