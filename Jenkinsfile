pipeline {
    agent any
    environment {
        PROJECT_ID = 'open-436911'
        CLUSTER_NAME = 'wimb-kube'
        LOCATION = 'us-central1-a'
        CREDENTIALS_ID = '25e628d7-2697-4d33-9d87-cf999794e66c'
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
                    cp sh "cp /var/jenkins_home/.env ./code/.env"
                }
            }
        }
        stage('Build image') {
            steps {
                script {
                    myapp = docker.build("crolvlee/wimb_test:${env.BUILD_ID}")
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
                sh "sed -i 's/wimb_test:latest/wimb_test:${env.BUILD_ID}/g' deployment.yaml"
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