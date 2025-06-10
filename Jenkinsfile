pipeline {
    agent { label 'local-build' }

    environment {
        ZIP_FILE = 'build.zip'
        SSH_PORT = '22'
        TEST_SERVER = 'ubuntu@jfrog:/home/ubuntu/artifactory/test'
        LIVE_SERVER = 'ubuntu@jfrog:/home/ubuntu/artifactory/main'
        ART_URL_BASE = 'http://192.168.137.111:8081/artifactory'
        TEST_ART_REPO_PATH = 'tester-web1/test/'
        LIVE_ART_REPO_PATH = 'tester-web1/live/'
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

        stage('Approval Before Cleanup') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME.startsWith('test/')) {
                        timeout(time: 24, unit: 'HOURS') {
                            input message: "Lanjut Clean?", submitter: 'devops-team'
                        }
                    } else {
                        echo "Branch '${env.BRANCH_NAME}' tidak memerlukan clean."
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
