pipeline {
    agent any

    stages {

        stage('Unit Tests') {
            steps {
                sh './run-unit-tests.sh'
            }
        }

        stage('Identify New Files') {
            agent {
                docker {
                    image 'ubuntu:noble'
                    args '-u root'
                    reuseNode true
                }
            }

            steps {
                script {
                    sh 'apt-get update && apt-get install -y git'

                    def addedFiles = sh(
                        script: "git diff --name-status HEAD~1 HEAD | awk '/^A/{print \$2}' | grep '^data-catalog-configs/'",
                        returnStdout: true
                    ).trim()

                    if (addedFiles) {
                        env.ADDED_FILES = addedFiles
                    } else {
                        echo 'No new files added.'
                        currentBuild.result = 'SUCCESS'
                    }
                }
            }
        }

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }

            environment {
                AWS_S3_BUCKET = 'data-catalog-lz'
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'tc-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 ls s3://$AWS_S3_BUCKET/OpenMetadata/
                    '''
                    script {
                        if (env.ADDED_FILES) {
                            def files = env.ADDED_FILES.split('\n')
                            for (file in files) {
                                sh "aws s3 cp ${file} s3://$AWS_S3_BUCKET/OpenMetadata/${file}"
                            }
                        } else {
                            echo "No new files found to upload."
                        }
                    }
                }
            }
        }
    }
}
