pipeline {
    agent any
    tools {nodejs "Node"}
    environment {
        CI = 'true'
        API_DIR = './TestStore'
        DEV_ENV = 'dev'
        LOCAL_ENV = 'host'
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
                echo 'Create a dev environment'
                sh '/home/apictl/apictl version'
                sh '/home/apictl/apictl remove env dev'
                sh '/home/apictl/apictl add-env -e dev --apim https://localhost:9443/ --token  https://localhost:8243/token --registration http://localhost:9763' 
                echo 'Logging into $DEV_ENV'
                withCredentials([usernamePassword(credentialsId: 'apim_dev', usernameVariable: 'DEV_USERNAME', passwordVariable: 'DEV_PASSWORD')]) {
                    sh '/home/apictl/apictl login $DEV_ENV -u $DEV_USERNAME -p $DEV_PASSWORD -k'                        
                }
                echo 'Deploying to $DEV_ENV'
                sh '/home/apictl/apictl import-api -f $API_DIR -e $DEV_ENV -k --preserve-provider --update --verbose'
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
                 echo "Create Host Env"
                sh '/home/apictl/apictl version'
                sh '/home/apictl/apictl remove env host'
                sh '/home/apictl/apictl add-env -e host --apim http://localhost:9443 --token  http://localhost:9443/token --registration http://localhost:9763'
                echo "Logging into $LOCAL_ENV"
                withCredentials([usernamePassword(credentialsId: 'apim_local', usernameVariable: 'LOCAL_USERNAME', passwordVariable: 'LOCAL_PASSWORD')]) {
                    sh '/home/apictl/apictl login $LOCAL_ENV -u $LOCAL_USERNAME -p $LOCAL_PASSWORD -k'                        
                }
                echo 'Deploying to Local'
                sh '/home/apictl/apictl import-api -f $API_DIR -e $LOCAL_ENV -k --preserve-provider --update --verbose'
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
