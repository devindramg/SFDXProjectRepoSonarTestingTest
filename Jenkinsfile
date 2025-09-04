pipeline {
    agent any

    environment {
        SONAR_URL = "http://192.168.1.100:9000"
        SONAR_ADMIN_USER = "admin"           // Sonar admin username
        SONAR_ADMIN_PASSWORD = "admin123"    // Sonar admin password
    }

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

        stage('Validate or Refresh SonarQube Token') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    powershell """
                        # Test existing token
                        try {
                            \$response = Invoke-RestMethod -Uri ${SONAR_URL}/api/system/status -Headers @{Authorization = "Basic \$([Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(''+\$env:SONAR_TOKEN+':')))"}
                            if (\$response.status -ne 'UP') { throw 'SonarQube server is not UP' }
                        } catch {
                            Write-Host "Existing SonarQube token invalid or server unreachable. Generating new token..."

                            # Generate a new token
                            \$body = @{name='jenkins-auto-token'}
                            \$newToken = Invoke-RestMethod -Uri ${SONAR_URL}/api/user_tokens/generate -Method Post -Body \$body -Credential (New-Object System.Management.Automation.PSCredential('${SONAR_ADMIN_USER}', (ConvertTo-SecureString '${SONAR_ADMIN_PASSWORD}' -AsPlainText -Force)))

                            Write-Host "New token generated: \$newToken.token"

                            # Optionally, update Jenkins credential using Jenkins CLI
                            # This requires jenkins-cli.jar on agent and proper permissions
                            # Example:
                            # java -jar jenkins-cli.jar -s http://localhost:8080/ create-credentials-by-xml system::system::jenkins _ < updated-credentials.xml

                            exit 0
                        }
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        bat """
                            C:\\sonar-scanner-7.2.0.5079-windows-x64\\bin\\sonar-scanner ^
                              -Dsonar.projectKey=SalesforceApp ^
                              -Dsonar.sources=force-app/main/default ^
                              -Dsonar.host.url=%SONAR_URL% ^
                              -Dsonar.login=%SONAR_TOKEN%
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
