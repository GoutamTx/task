pipeline {
 
    agent any
    environment {
 
        ARTIFACTORY_URL = 'https://trialmewlv1.jfrog.io/artifactory'
 
        ARTIFACTORY_REPO = 'taskrepo-generic-local'
 
        ARTIFACTORY_PATH = "app/${BUILD_NUMBER}"
 
        ARTIFACTORY_CREDENTIALS_ID = 'jfrog-creds'
 
        ARTIFACT_NAME = 'artifact.zip'
        EC2_INSTANCE_IP = '52.1.23.80'
        EC2_USER = 'ec2-user' // Adjust based on your EC2 instance's user
        SSH_KEY_PATH = '/home/gautam/my_new_key.pem' // Path to your private SSH key
 
 
    }
    stages {
 
        stage('Checkout Code') {
 
            steps {
 
                git branch: 'main', url: 'https://github.com/GoutamTx/task.git'
 
            }
 
        }
        stage('Archive Code') {
 
            steps {
 
                sh '''
 
                echo "Creating artifact.zip..."
 
                zip -r ${ARTIFACT_NAME} . -x ".git/*" ".git" "*.log" "*.tmp" "*jenkins*" || exit 1
 
                '''
 
            }
 
        }
        stage('Upload to Artifactory') {
 
            steps {
 
                withCredentials([usernamePassword(
 
                    credentialsId: "${env.ARTIFACTORY_CREDENTIALS_ID}",
 
                    usernameVariable: 'ART_USER',
 
                    passwordVariable: 'ART_PASS'
 
                )]) {
 
                    sh '''
 
                    echo "Uploading ${ARTIFACT_NAME} to Artifactory..."
 
                    curl -u "$ART_USER:$ART_PASS" -T "${ARTIFACT_NAME}" "$ARTIFACTORY_URL/$ARTIFACTORY_REPO/$ARTIFACTORY_PATH/${ARTIFACT_NAME}"
 
                    '''
 
                }
 
            }
 
        }
       stage('Deploying on EC2') {
            steps {
                withCredentials([
                    file(credentialsId: 'ec2-user', variable: 'SSH_KEY'),
                    usernamePassword(credentialsId: 'jfrog-creds', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')
                ]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no -i \$SSH_KEY ${EC2_USER}@${EC2_INSTANCE_IP} '
                        ART_USER="${ART_USER}"
                        ART_PASS="${ART_PASS}"
                        sudo apt update -y
                        sudo apt install -y apache2 unzip curl
                        cd /tmp
                        wget --user="\${ART_USER}" --password="\${ART_PASS}" -O ${ARTIFACT_NAME} "${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}/${ARTIFACTORY_PATH}/${ARTIFACT_NAME}"
                        sudo unzip -o ${ARTIFACT_NAME} -d /var/www/html/
                        sudo systemctl restart apache2
                    '
                """
                }
            }
       }
    }
    post {
 
        success {
 
            echo "✅ Uploaded artifact.zip to Artifactory successfully!"
 
        }
 
        failure {
 
            echo "❌ Failed to upload artifact."
 
        }
 
    }
 
}
 
