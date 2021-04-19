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
                url: 'https://github.com/Atwinenickson/ws02',
                credentialsId: 'root'
            }
        }
        stage('Deploy to Dev') {
            environment{
                RETRY = '80'
            }
            steps {
                echo 'Create a dev environment'
                sh '/home/atwine/Pictures/apictl/apictl list envs'
                sh '/home/atwine/Pictures/apictl/apictl add-env -e dev --apim https://localhost:9444'
                echo 'Logging into $DEV_ENV'
                withCredentials([usernamePassword(credentialsId: 'apim_dev', usernameVariable: 'DEV_USERNAME', passwordVariable: 'DEV_PASSWORD')]) {
                    sh '/home/atwine/Pictures/apictl/apictl login $DEV_ENV -u $DEV_USERNAME -p $DEV_PASSWORD -k'                        
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
        
           stage('Deploy to Local') {
            environment{
                RETRY = '80'
            }
            steps {
                echo 'Create a local environment'
                sh '/home/atwine/Pictures/apictl/apictl list envs'
                sh '/home/atwine/Pictures/apictl/apictl add-env -e host --apim https://localhost:9444'
                echo 'Logging into $LOCAL_ENV'
                withCredentials([usernamePassword(credentialsId: 'apim_dev', usernameVariable: 'LOCAL_USERNAME', passwordVariable: 'LOCAL_PASSWORD')]) {
                    sh '/home/atwine/Pictures/apictl/apictl login $LOCAL_ENV -u $LOCAL_USERNAME -p $LOCAL_PASSWORD -k'                        
                }
                echo 'Deploying to $LOCAL_ENV'
                sh 'apictl import-api -f $API_DIR -e $DEV_ENV -k --preserve-provider --update --verbose'
            }
        }
    }
}
