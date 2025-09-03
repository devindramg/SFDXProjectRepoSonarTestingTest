pipeline {
    agent any

    tools {
        sonar 'SonarScanner'   // Use the SonarScanner tool configured in Jenkins
    }

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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    bat """
                        ${tool 'SonarScanner'}\\bin\\sonar-scanner ^
                            -Dsonar.projectKey=SFDXProject ^
                            -Dsonar.sources=force-app ^
                            -Dsonar.host.url=http://localhost:9000
                    """
                }
            }
        }
    }
}
