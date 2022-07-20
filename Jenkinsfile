pipeline {
    agent any
    environment{  //هذي البيئه سواء سوينا اختصار او لا ماتفرق بس عشان مانجلس كل مرة نكتبها نستدعيها بالدولار  
        AWS_S3_BUCKET = "pipeline-eb"
        ARTIFACT_NAME = "hello-world.war"
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
        AWS_EB_APP_NAME = "pipeline"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Pipeline-env"
    }

    stages {
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
                always{
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        stage('Package') {
            steps {
                sh "mvn package"
                
            }
            post{
                success{
                    archiveArtifacts artifacts: '**/target/**.war', followSymlinks: false
                }
            }
        }
    
        stage('publish artfacts to s3') {
            steps {
                sh "aws configure set region us-east-1" //عشان يعرف مكان البوكت فين 
                sh "aws s3 cp ./target/**.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"//ينسخ ملف الوار اللي اساسا جهزناه بالميفن من السيرفر الى اس٣
            }
        }

        stage('Deploy') {
            steps {

                sh "aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME"

                sh "aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION"
            }
        }
    }
}
