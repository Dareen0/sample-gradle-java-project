pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('dareen-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('dareen-aws-secret-access-key')
        AWS_S3_BUCKET = "sample-proj"
        ARTIFACT_NAME = "sample-gradle.jar"
        AWS_EB_APP_NAME = "gradle-project"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Gradleproject-env"
    }
    stages {
        stage('Validate') {
            steps {
                sh "chmod +x gradlew"
                sh "./gradlew clean"
            }
        }
        stage('Build') {
            steps {
                sh "./gradlew build"
                sh "./gradlew assemble"

            }
        }
        stage('Test') {
            steps {
                sh "./gradlew test"
            }
        }
        stage('Package') {
            steps {
                sh "./gradlew jar"
            }
            post{
                success{
                    archiveArtifacts artifacts: 'build/libs/samplegradle-0.0.1-SNAPSHOT.jar', followSymlinks: false
                }
            }
        }
        stage('Publish artifacts to S3 Bucket') {
            steps {
                sh "aws configure set region us-east-2"
                sh "aws s3 cp ./build/libs/samplegradle-0.0.1-SNAPSHOT.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
            }
         }
        stage('Deploy') {
            steps {
                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
         }
        

    }
}