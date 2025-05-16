pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'https://trialx3clu6.jfrog.io/artifactory'
        ARTIFACTORY_REPO = 'firstrepo'
        ARTIFACT_NAME = 'artifact'
        EC2_INSTANCE_IP = '52.1.23.80'
        EC2_USER = 'ec2-user'
        SSH_KEY_PATH = '/var/lib/jenkins/.ssh/webserver.pem'
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
                sh 'zip -r artifact.zip website.html build/* || true'
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

        stage('Deploy on EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ${EC2_USER}@${EC2_INSTANCE_IP} <<EOF
                    sudo yum install -y httpd
                    wget --user="${ART_USER}" --password="${ART_PASS}" -O /tmp/${ARTIFACT_NAME} "$ARTIFACTORY_URL/$ARTIFACTORY_REPO/$ARTIFACTORY_PATH/${ARTIFACT_NAME}"
                    sudo unzip -o artifact.zip -d /var/www/html/
                    sudo systemctl restart httpd
                EOF
                """
            }
        }
    }
}
