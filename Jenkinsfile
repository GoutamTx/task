pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'https://trialx3clu6.jfrog.io/artifactory'
        ARTIFACTORY_REPO = 'firstrepo'
        ARTIFACT_NAME = 'artifact'
        EC2_INSTANCE_IP = '3.91.252.181'
        EC2_USER = 'ec2-user' // Adjust based on your EC2 instance's user
        SSH_KEY_PATH = '/var/lib/jenkins/.ssh/webserver.pem' // Path to your private SSH key
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-creds-id', url: 'https://github.com/GoutamTx/task.git'
            }
        }

        stage('Package') {
    steps {
        sh '''
        zip -r artifact.zip website.html build/* || true
        '''
    }
}

        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                    sh """
                        curl -u $ART_USER:$ART_PASS -T ${ARTIFACT_NAME}.zip \
    "${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}/myapp/${BUILD_NUMBER}/${ARTIFACT_NAME}.zip"

                    """
                }
            }
        }

     stage('depoying on Ec2'){
         steps{
             sh '''
             ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ${EC2_USER}@${EC2_INSTANCE_IP} <<EOF
             sudo yum install -y httpd
            cd /tmp &&
            curl -u $ART_USER:$ART_PASS -O "${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}/myapp/${BUILD_NUMBER}/artifact.zip" &&
            sudo unzip -o artifact.zip -d /var/www/html/
            sudo systemctl restart httpd
            EOF
             '''
         }
     }
    }
}
