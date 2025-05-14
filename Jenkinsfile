pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'https://trialx3clu6.jfrog.io/artifactory'
        ARTIFACTORY_REPO = 'firstrepo'
        ARTIFACT_NAME = 'artifact'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-creds-id', url: 'https://github.com/GoutamTx/task.git'
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Build ${BUILD_NUMBER}" > build/${ARTIFACT_NAME}'
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                    sh """
                        curl -u $ART_USER:$ART_PASS -T build/${ARTIFACT_NAME} \
                        "${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}/myapp/${BUILD_NUMBER}/${ARTIFACT_NAME}"
                    """
                }
            }
        }
    }
}
