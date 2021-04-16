pipeline {
    agent any
    tools {nodejs "Node"}
    environment {
        CI = 'true'
        API_DIR = './TestStore'
        DEV_ENV = 'dev'
        LOCAL_ENV = 'local'
        TEST_SCRIPT_FILE = 'sample.store.dev.postman_collection.json'        
    }
    stages {
        stage('Preparation') {
            steps{
                git branch: "master",
                url: 'http://192.168.0.82:4000/root/jenkinspipeline',
                credentialsId: 'root'
            }
        }
        stage('Deploy to Dev') {
            environment{
                RETRY = '80'
            }
            steps {
                echo 'Logging into $DEV_ENV'
                withCredentials([usernamePassword(credentialsId: 'root', usernameVariable: 'DEV_USERNAME', passwordVariable: 'DEV_PASSWORD')]) {
                    sh 'apictl login $DEV_ENV -u $DEV_USERNAME -p $DEV_PASSWORD -k'                        
                }
                echo 'Deploying to $DEV_ENV'
                sh 'apictl import-api -f $API_DIR -e $DEV_ENV -k --preserve-provider --update --verbose'
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Running tests in $DEV_ENV'
                sh 'newman run $API_DIR/$TEST_SCRIPT_FILE --insecure' 
            }
        }
        stage('Deploy to Local Environment') {
            environment{
                RETRY = '60'
            }
            steps {
                sh 'echo "Logging into $LOCAL_ENV"'
                withCredentials([usernamePassword(credentialsId: 'root', usernameVariable: 'PROD_USERNAME', passwordVariable: 'PROD_PASSWORD')]) {
                    sh 'apictl login $LOCAL_ENV -u $PROD_USERNAME -p $PROD_PASSWORD -k'                        
                }
                echo 'Deploying to Production'
                sh 'apictl import-api -f $API_DIR -e $LOCAL_ENV -k --preserve-provider --update --verbose'
            }
        }
    }
    post {
        cleanup {
            deleteDir()
            dir("${workspace}@tmp") {
                deleteDir()
            }
            dir("${workspace}@script") {
                deleteDir()
            }
        }
    }
}
