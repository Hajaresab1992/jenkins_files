pipeline {
    agent any

    environment {
        AWS_PROFILE = 's3_artifacts'  // Use your configured AWS CLI profile
        S3_BUCKET = 'artifacts-pipeline-script'
        REGION = 'ap-south-1'
    }

    stages {

        stage('Stage-1: Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                cleanWs()
            }
        }

        stage('Stage-2: Git Clone') {
            steps {
                echo 'Cloning Git repository...'
                git branch: 'main', url: '[https://github.com/Hajaresab1992/jenkins_files.git]'
            }
        }

        stage('Stage-3: MVN Install') {
            steps {
                echo 'Installing MVN dependencies...'
                sh 'mvn install'
            }
        }

        stage('Stage-4: Archive Artifact (tar.gz)') {
            steps {
                script {
                    echo 'Archiving files...'
                    sh """
                        mkdir -p /tmp/archive
                        rsync -av --exclude='.git' ./ /tmp/archive/
                        tar -czvf artifact-${JOB_NAME}-${BUILD_NUMBER}-${BUILD_ID}.tar.gz -C /tmp/archive .
                    """
                }
            }
        }

        stage('Stage-5: Upload Artifact to S3') {
            steps {
                echo 'Uploading artifact to S3...'
               s3Upload(
                  bucket: 'your-s3-bucket-name',
                  path: 'your/s3/path/',
                  file: 'your-local-file.txt',
                  workingDir: '.',
                  dontWaitForConcurrentBuildCompletion: true
                    )

            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
