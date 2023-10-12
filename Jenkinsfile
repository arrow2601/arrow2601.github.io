node {
    def gitRepo = 'https://github.com/arrow2601/arrow2601.github.io.git'
    def serverUsername = 'rehan'
    def serverIP = '172.18.0.1'
    def serverDestination = '/var/www/html'
    def sshPassword = '1234'
    
    stage('Checkout') {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: gitRepo]]])
    }

    stage('Build') {
        docker.image('python:3').inside {
            sh 'pip install mkdocs'
            sh 'mkdocs build'
            archiveArtifacts artifacts: 'site/**', allowEmptyArchive: true
        }
    }

    stage('Deploy') {
        step([$class: 'ArtifactArchiver', artifacts: 'site/**'])
        sh "sshpass -p '${sshPassword}' scp -o StrictHostKeyChecking=no -r site ${serverUsername}@${serverIP}:${serverDestination}"
    }
    
    post {
        always {
            // Clean up if needed
        }
    }
}

