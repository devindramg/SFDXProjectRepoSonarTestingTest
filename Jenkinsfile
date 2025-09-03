pipeline {
    agent any

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/devindramg/SFDXProjectPrivateRepoTest.git',
                    credentialsId: 'my-github-token'
            }
        }

        stage('Authenticate Salesforce Org') {
            steps {
                withCredentials([file(credentialsId: 'sf-jwt-key', variable: 'JWT_KEY_FILE')]) {
                    bat '''
                    echo Authorizing with Salesforce...

                    sf org login jwt ^
                        --client-id 3MVG99AeQQhMVo3SAh7nBX_evEGcaF3nEdNhkHZ35.h2DnvCcEwXZBuqdPoAw1q7hOkUoSSJ08eAwHUOoS.jt ^
                        --jwt-key-file %JWT_KEY_FILE% ^
                        --username ktdocsgendevorg@techkasetti.com ^
                        --instance-url https://login.salesforce.com ^
                        --alias DevOrg

                    echo Authentication Successful!
                    '''
                }
            }
        }

        stage('Deploy to Salesforce') {
            steps {
                bat '''
                echo Deploying metadata to Salesforce...
                sf project deploy start --target-org DevOrg --verbose --wait 10
                '''
            }
        }
    }
}
