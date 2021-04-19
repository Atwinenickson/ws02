pipeline {
    agent any
    tools {nodejs "Node"}
    environment {
        CI = 'true'
        API_DIR = '/var/lib/jenkins/workspace/WSO2'
        DEV_ENV = 'dev'
        LOCAL_ENV = 'host'
        TEST_SCRIPT_FILE = 'sample_store.postman_collection.json'        
    }
    stages {
        stage('Preparation') {
            steps{
                git branch: "master",
                url: 'https://github.com/Atwinenickson/ws02.git',
                credentialsId: 'root'
            }
        }
        stage('Deploy to Dev') {
            environment{
                RETRY = '80'
            }
            steps {
                echo 'Create a dev environment'
                sh '/home/atwine/Pictures/apictl/apictl remove env dev'
                sh '/home/atwine/Pictures/apictl/apictl list envs'
                sh '/home/atwine/Pictures/apictl/apictl add-env -e dev --apim https://localhost:9444'
                echo 'Logging into $DEV_ENV'
                withCredentials([usernamePassword(credentialsId: 'apim_dev', usernameVariable: 'DEV_USERNAME', passwordVariable: 'DEV_PASSWORD')]) {
                    sh '/home/atwine/Pictures/apictl/apictl login $DEV_ENV -u $DEV_USERNAME -p $DEV_PASSWORD -k'                        
                }
                echo 'Deploying to $DEV_ENV'
                sh '/home/atwine/Pictures/apictl/apictl import-api -f $API_DIR -e $DEV_ENV -k --preserve-provider --update --verbose'
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Running tests in $DEV_ENV'
                sh 'newman run $API_DIR/$TEST_SCRIPT_FILE --insecure' 
            }
        }
    }
}
