pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal-demo1'
        RESOURCE_GROUP = 'python-jenkins-rg'
        APP_SERVICE_NAME = 'python-jenkins-webapp-service'
        PYTHON_VERSION = '3.12'
        PYTHON_PATH = 'C:\\Users\\DELL\\AppData\\Local\\Programs\\Python\\Python312\\python'
        AZ_CLI_PATH = 'C:\\Program Files\\Microsoft SDKs\\Azure\\CLI2\\wbin'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                 git branch: 'master', url: 'https://github.com/Atishay-Jain01/Python-Jenkins-demo.git'
            }
        }
        
        stage('Build') {
            steps {
                bat '''
                    "%PYTHON_PATH%" --version
                    "%PYTHON_PATH%" -m pip install --upgrade pip
                    "%PYTHON_PATH%" -m pip install -r requirements.txt
                '''
            }
        }

        stage('Debug az') {
            steps {
                bat '''
                    set PATH=C:\\Program Files\\Microsoft SDKs\\Azure\\CLI2\\wbin;%AZ_CLI_PATH%
                    echo PATH is: %AZ_CLI_PATH%
                    where az
                    az.cmd --version
                '''
            }
        }

        
        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat '''
                        set PATH=C:\\Program Files\\Microsoft SDKs\\Azure\\CLI2\\wbin;%AZ_CLI_PATH%
                        "az.cmd" login --service-principal -u "%AZURE_CLIENT_ID%" -p "%AZURE_CLIENT_SECRET%" --tenant "%AZURE_TENANT_ID%"
                        "az.cmd" group create --name %RESOURCE_GROUP% --location eastus
                        "az.cmd" appservice plan create --name %APP_SERVICE_NAME%-plan --resource-group %RESOURCE_GROUP% --sku B1 --is-linux
                        "az.cmd" webapp create --resource-group %RESOURCE_GROUP% --plan %APP_SERVICE_NAME%-plan --name %APP_SERVICE_NAME% --runtime "PYTHON|%PYTHON_VERSION%"
                        "az.cmd" webapp config set --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --startup-file "gunicorn --bind=0.0.0.0 --timeout 600 app:app"
                        powershell Compress-Archive -Path ./* -DestinationPath ./deploy.zip -Force
                        dir "%cd%\\deploy.zip"
                        "az.cmd" webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path ./deploy.zip --type zip
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
