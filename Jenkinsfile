pipeline {
    agent { label 'local-build' }

    environment {
        ZIP_FILE = 'build.zip'
        SSH_PORT = '22'

        TEST_HOST = 'ubuntu@jfrog'
        TEST_PATH = '~/artifactory/test'

        LIVE_HOST = 'ubuntu@jfrog'
        LIVE_PATH = '~/artifactory/main'

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

        stage('Send ZIP') {
            steps {
                script {
                    def targetHost = null
                    def targetPath = null

                    if (env.BRANCH_NAME == 'main') {
                        targetHost = env.LIVE_HOST
                        targetPath = env.LIVE_PATH
                    } else if (env.BRANCH_NAME.startsWith('test/')) {
                        targetHost = env.TEST_HOST
                        targetPath = env.TEST_PATH
                    }

                    if (targetHost && targetPath) {
                        echo "Deploying ZIP to ${targetHost}:${targetPath}"
                        withCredentials([usernamePassword(credentialsId: 'jfrog-ssh', usernameVariable: 'SSH_USER', passwordVariable: 'SSH_PASS')]) {
                            def hostOnly = targetHost.split('@')[-1]
                            sh """
                                cd ~/mystore
                                sshpass -p "$SSH_PASS" rsync -avzhp -e "ssh -o StrictHostKeyChecking=yes -p ${env.SSH_PORT}" ${env.ZIP_FILE} "$SSH_USER@${hostOnly}:${targetPath}"
                            """
                        }
                    } else {
                        echo "No valid deployment target for branch: ${env.BRANCH_NAME}"
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

        stage('Approval Auto Deployment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME.startsWith('test/')) {
                        timeout(time: 24, unit: 'HOURS') {
                            input message: "Are you sure you want to proceed with the auto-deployment process from this branch '${env.BRANCH_NAME}'?", submitter: 'devops-team'
                        }
                    } else {
                        echo "Branch '${env.BRANCH_NAME}' tidak memerlukan approval."
                    }
                }
            }
        }

        stage('Auto Deployment') {
            steps {
                script {
                    def targetHost = null

                    if (env.BRANCH_NAME == 'main') {
                        targetHost = env.LIVE_HOST
                    } else if (env.BRANCH_NAME.startsWith('test/')) {
                        targetHost = env.TEST_HOST
                    }

                    if (targetHost) {
                        echo "Menjalankan perintah 'hostname' di ${targetHost}"
                        withCredentials([usernamePassword(credentialsId: 'my-ssh-password', usernameVariable: 'SSH_USER', passwordVariable: 'SSH_PASS')]) {
                            def hostOnly = targetHost.split('@')[-1]
                            sh """
                                sshpass -p "$SSH_PASS" ssh -o StrictHostKeyChecking=yes -p ${env.SSH_PORT} "$SSH_USER@${hostOnly}" 'hostname'
                            """
                        }
                    } else {
                        echo "Tidak ada server deployment untuk branch ini."
                    }
                }
            }
        }
    }
}
