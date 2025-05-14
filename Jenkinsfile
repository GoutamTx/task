pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'https://trialx3clu6.jfrog.io/artifactory'
        ARTIFACTORY_REPO = 'firstrepo'
        ARTIFACT_NAME = 'artifact.zip'
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

        stage('Build') {
            steps {
                sh '''
                    mkdir -p build
                    rsync -av --exclude=build/ ./ build/
                '''
            }
        }

        stage('Package') {
            steps {
                sh '''
                    cd build
                    if [ "$(ls -A)" ]; then
                        zip -r ../${ARTIFACT_NAME} ./*
                    else
                        echo "❌ Nothing to zip. 'build/' directory is empty."
                        exit 1
                    fi
                '''
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                    sh '''
                        echo "Uploading artifact to Artifactory..."
                        curl -u $ART_USER:$ART_PASS -T ${ARTIFACT_NAME} \
                        "${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}/myapp/${BUILD_NUMBER}/${ARTIFACT_NAME}"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Artifact uploaded successfully to Artifactory."
        }
        failure {
            echo "❌ Build or upload failed."
        }
    }
}
