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
