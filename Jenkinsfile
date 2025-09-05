pipeline {
    agent any

    environment {
        SONAR_URL   = "http://localhost:9000"
        SONAR_TOKEN = "your_hardcoded_sonar_token_here"  // or use Jenkins credentials if preferred
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'my-github-token',
                    url: 'https://github.com/devindramg/SFDXProjectRepoSonarTestingTest.git'
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

        stage('SonarQube Check & Analysis') {
            steps {
                script {
                    def sonarReachable = false
                    try {
                        // Check if SonarQube is UP
                        def response = powershell(returnStdout: true, script: """
                            try {
                                (Invoke-RestMethod -Uri ${env.SONAR_URL}/api/system/status -UseBasicParsing).status
                            } catch { "DOWN" }
                        """).trim()

                        echo "SonarQube status: ${response}"
                        sonarReachable = (response == "UP")
                    } catch (err) {
                        echo "SonarQube check failed: ${err}"
                        sonarReachable = false
                    }

                    if (sonarReachable) {
                        withSonarQubeEnv('MySonarQubeServer') {
                            bat """
                                C:\\sonar-scanner-7.2.0.5079-windows-x64\\bin\\sonar-scanner ^
                                  -Dsonar.projectKey=SalesforceApp ^
                                  -Dsonar.sources=force-app/main/default ^
                                  -Dsonar.host.url=${env.SONAR_URL} ^
                                  -Dsonar.login=${env.SONAR_TOKEN}
                            """
                        }
                    } else {
                        echo "⚠ SonarQube not reachable → skipping analysis."
                        currentBuild.result = 'UNSTABLE'   // Mark build as yellow if skipped
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    try {
                        timeout(time: 2, unit: 'MINUTES') {
                            waitForQualityGate abortPipeline: false
                        }
                    } catch (err) {
                        echo "Skipping Quality Gate check because SonarQube is not reachable."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
    }
}
