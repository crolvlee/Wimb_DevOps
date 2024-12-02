node {
    def app
    stage('Clone repository') {
        git 'https://github.com/crolvlee/Wimb_DevOps.git'
    }
    stage('Build image') {
        app = docker.build('crolvlee/wimb_test')
    }
    stage('Push image') {
        docker.withRegistry('https://registry.hub.docker.com', 'crolvlee') {
            app.push("${env.BUILD_NUMBER}")
            app.push('latest')
        }
    }
}