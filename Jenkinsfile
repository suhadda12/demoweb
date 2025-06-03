pipeline {
    agent { label 'local-build' }

    environment {
        ZIP_FILE = 'build.zip'
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

        stage('Checking Encryption') {
            steps {
                script {
                    echo 'Checking if git-crypt is used...'
                    sh '''
                        if command -v git-crypt >/dev/null 2>&1; then
                            echo "git-crypt found. Checking status..."
                            git-crypt status -e || true
                            echo "Attempting to unlock..."
                            git-crypt unlock || true
                        else
                            echo "git-crypt not found, skipping unlock."
                        fi
                    '''
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
                '''
            }
        }

        stage('Done') {
            steps {
                echo "ZIP file created: ${env.ZIP_FILE}"
                sh 'ls -lh "$ZIP_FILE"'
            }
        }
    }
}
