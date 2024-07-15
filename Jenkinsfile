pipeline {
    agent any

    environment {
        DEV_SERVER = "ec2-user@35.86.80.247"
        ARTIFACT_NAME = 'nodejs-app.tar.gz'
    }

    stages {
        stage('Build') {
            steps {
                script {
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
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key-aws', keyFileVariable: 'SSH_KEY')]) {
                    script {
                        deployToServer(DEV_SERVER, SSH_KEY)
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

def deployToServer(server, sshKey) {
    sh """
        set -e
        mkdir -p ~/.ssh
        ssh-keyscan -H ${server.split('@')[1]} >> ~/.ssh/known_hosts
        scp -i ${sshKey} ${ARTIFACT_NAME} ${server}:/tmp/ || exit 1
        ssh -i ${sshKey} ${server} 'tar -xzvf /tmp/${ARTIFACT_NAME} -C /tmp/' || exit 1
        ssh -i ${sshKey} ${server} '/usr/bin/npm install --prefix /tmp' || exit 1
        ssh -i ${sshKey} ${server} 'nohup /usr/bin/npm start --prefix /tmp/app.js > /tmp/nodejs-app.log 2>&1 &' || exit 1
    """
}
