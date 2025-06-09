pipeline {
    agent { label 'local-build' }

    environment {
        ZIP_FILE = 'build.zip'
        SSH_PORT = '22'
        TEST_SERVER = 'ubuntu@jfrog:/home/ubuntu/artifactory/test'
        LIVE_SERVER = 'ubuntu@jfrog:/home/ubuntu/artifactory/main'
        ART_URL_BASE = 'http://192.168.137.111:8081/artifactory'  // ✅ tanpa trailing slash
        TEST_ART_REPO_PATH = 'tester-web1/test/'                 // ✅ dengan repo key
        LIVE_ART_REPO_PATH = 'tester-web1/live/'                 // ✅ dengan repo key
    }

    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm
                echo "Checked out branch: ${env.BRANCH_NAME}"
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
                        -x ".git/*" "Jenkinsfile" ".workflow"
                    mkdir -p ~/mystore
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
                        echo "Deploying to: ${targetServer}"
                        sh """
                            cd ~/mystore
                            rsync -avzhp -e "ssh -p ${env.SSH_PORT}" ${env.ZIP_FILE} "${targetServer}"
                        """
                    } else {
                        echo "No valid deployment target for branch: ${env.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                script {
                    def repoPath = null
                    if (env.BRANCH_NAME == 'main') {
                        repoPath = env.LIVE_ART_REPO_PATH
                    } else if (env.BRANCH_NAME.startsWith('test/')) {
                        repoPath = env.TEST_ART_REPO_PATH
                    }

                    if (repoPath) {
                        echo "Uploading to: ${env.ART_URL_BASE}/${repoPath}${env.ZIP_FILE}"

                        withCredentials([usernamePassword(credentialsId: 'myjfrog', usernameVariable: 'ART_USERNAME', passwordVariable: 'ART_API_TOKEN')]) {
                            sh """
                                cd ~/mystore
                                curl -u "\$ART_USERNAME:\$ART_API_TOKEN" -T "\$ZIP_FILE" "${env.ART_URL_BASE}/${repoPath}${env.ZIP_FILE}"
                            """
                        }
                    } else {
                        echo "Skipping Artifactory upload: unsupported branch ${env.BRANCH_NAME}"
                    }
                }
            }
        }



        stage('Cleanup store directory') {
            steps {
                echo "Cleaning up ZIP file from ~/mystore"
                sh 'rm -f ~/mystore/build.zip'
            }
        }
    }
}
