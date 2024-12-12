pipeline {
    agent any

    stages {

        stage('Unit Tests') {
            steps {
                sh './run-unit-tests.sh'
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
                        echo "Hello S3!" > hello.txt
                        aws s3 cp hello.txt s3://$AWS_S3_BUCKET/OpenMetadata/
                    '''
                }
            }
        }
    }
}
