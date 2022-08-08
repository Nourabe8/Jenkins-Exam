pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('NouraAlotaibi-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('NouraAlotaibi-aws-secret-access-key')
        NOURA_RUNNING_APP = credentials('NouraAlotaibi-running-app')
        ARTIFACT_NAME = 'samplejava.jar'
        AWS_S3_BUCKET = 'nouralotaibi-belt2d2-artifacts-123456'
        AWS_EB_APP_NAME = 'NouraAlotaibi_EB'
        AWS_EB_ENVIRONMENT = 'Nouraalotaibieb-env-1'
        AWS_EB_APP_VERSION = "${BUILD_ID}"
    } 
    stages {

        stage('Quality Scan') {
            steps {
               
            sh "mvn -Dmaven.test.failure.ignore=true clean compile"
               
            sh "mvn clean verify sonar:sonar \
  -Dsonar.projectKey=onsite-NouraAlotaibi-B2D2 \
  -Dsonar.host.url=http://52.23.193.18 \
  -Dsonar.login=sqp_20cbc81cf967c6a018a3502031d788113db63874"

               
            }
            
        }

        stage('Validate') {
            steps {
                
                sh "mvn validate"

                sh "mvn clean"

            }
        }

         stage('Build') {
            steps {
                
                sh "mvn compile"

            }
        }

        stage('Test') {
            steps {
                
                sh "mvn test"

            }

            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }

        stage('Package'){
            steps {
                
                sh "mvn package"
            }
              post {
                success {
                    archiveArtifacts artifacts: '/target/*', followSymlinks: false
                    sh 'aws configure set region us-east-1'
                    sh 'aws s3 cp ./target/* s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
                }
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