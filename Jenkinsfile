pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/devindramg/SFDXProjectRepoSonarTestingTest.git',
                    credentialsId: 'my-github-token'
            }
        }

        stage('Authenticate Salesforce Org') {
            steps {
                withCredentials([string(credentialsId: 'sfdx-auth-url', variable: 'SFDX_AUTH_URL')]) {
                    bat """
                        echo %SFDX_AUTH_URL% > auth.txt
                        sf org login sfdx-url --sfdx-url-file auth.txt -a MyDevOrg --set-default
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    bat """
                        C:\\sonar-scanner-7.2.0.5079-windows-x64\\bin\\sonar-scanner ^
                          -Dsonar.projectKey=SalesforceApp ^
                          -Dsonar.sources=force-app/main/default ^
                          -Dsonar.host.url=http://<SONAR_SERVER>:9000 ^
                          -Dsonar.token=YOUR_SONAR_TOKEN
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Quality Gate failed with status: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
