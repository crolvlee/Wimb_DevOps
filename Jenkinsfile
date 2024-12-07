pipeline {
    agent any
    environment {
        PROJECT_ID = 'sswu-wimb-project'
        CLUSTER_NAME = 'wimb-sql-kube'
        LOCATION = 'us-central1-f'
        CREDENTIALS_ID = '01cfbed5-8dad-4a60-a53d-7cb802966f30'
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
                    sh "cp /var/jenkins_home/wimb-sql.env ./code/.env"
                }
            }
        }
        stage('Build image') {
            steps {
                script {
                    myapp = docker.build("crolvlee/wimb_sql:${env.BUILD_ID}")
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
				branch 'sql-test'
			}
            steps{
                sh "sed -i 's/wimb_sql:latest/wimb_sql:${env.BUILD_ID}/g' deployment.yaml"
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