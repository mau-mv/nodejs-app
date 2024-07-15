pipeline {
    agent any

    environment {
        DEV_SERVER = "ec2-user@35.86.80.247"
        SSH_KEY = credentials("ssh-key-aws")
        ARTIFACT_NAME = 'nodejs-app.tar.gz'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Install npm globally (if needed)
                    sh 'npm install -g npm@latest'
                    // Install dependencies
                    sh 'npm install'
                    // Package the application into a tarball
                    sh """
                        tar -czvf ${env.ARTIFACT_NAME} app.js package.json
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    sshagent(credentials: ['ssh-key-aws']){
                        deployToServer(DEV_SERVER)
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

def deployToServer(server) {
    sh """
        mkdir -p ~/.ssh
        ssh-keyscan -H ${server.split('@')[1]} >> ~/.ssh/known_hosts
        scp -i ${SSH_KEY} ${ARTIFACT_NAME} ${server}:/tmp/
        ssh -i ${SSH_KEY} ${server} 'tar -xzvf /tmp/${ARTIFACT_NAME} -C /tmp/'
        ssh -i ${SSH_KEY} ${server} 'npm install --prefix /tmp'
        ssh -i ${SSH_KEY} ${server} 'nohup npm start --prefix /tmp/app.js > /tmp/nodejs-app.log 2>&1 &'
    """
}
