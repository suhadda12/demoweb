pipeline {
    agent { label 'local-build' }

    environment {
        ZIP_FILE = 'build.zip'
        TEST_SERVER = 'ubuntu@jfrog:/home/ubuntu/artifactory/test'
        LIVE_SERVER = 'ubuntu@jfrog:/home/ubuntu/artifactory/main'
        SSH_PORT = '22'
    }

    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm
                echo "Checked out branch: ${env.BRANCH_NAME}"
                echo 'Listing all directories and files...'
                sh 'ls -la'
            }
        }

        stage('Check Encryption') {
            steps {
                script {
                    echo 'Checking encrypted files...'
                    sh 'git-crypt status -e || exit 0'
                }
            }
        }

        stage('Create ZIP') {
            steps {
                echo "Creating ZIP file: ${env.ZIP_FILE}"
                sh '''
                    zip -r "$ZIP_FILE" . \
                    -x ".git/*" \
                       "Jenkinsfile" \
                       ".workflow"
                    mv "$ZIP_FILE" ~/mystore/
                '''
            }
        }

        stage('Deploy ZIP') {
            steps {
                script {
                    def targetServer = null

                    if (env.BRANCH_NAME == 'main') {
                        targetServer = env.LIVE_SERVER
                    } else if (env.BRANCH_NAME.startsWith('test/')) {
                        targetServer = env.TEST_SERVER
                    }

                    if (targetServer) {
                        echo "Deploying ~/mystore/${env.ZIP_FILE} to ${targetServer}"
                        sh """
                            rsync -avzhp -e "ssh -p ${env.SSH_PORT}" ~/mystore/${env.ZIP_FILE} "${targetServer}"
                        """
                    } else {
                        echo "No deployment target for branch: ${env.BRANCH_NAME}. Skipping deployment."
                    }
                }
            }
        }

        stage('Cleanup store directory') {
            steps {
                echo "ZIP file created and moved to ~/mystore:"
                sh 'ls -la ~/mystore/build.zip'
                sh 'rm -fr ~/mystore/build.zip'
            }
        }
    }
}
