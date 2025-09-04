pipeline {
    agent any

    environment {
        SFDX_AUTH_URL = credentials('sfdx-auth-url') // Store your auth URL in Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/devindramg/SFDXProjectRepoSonarTestingTest.git',
                    credentialsId: 'my-github-token'
            }
        }

        stage('SFDX Auth') {
            steps {
                bat """
                    echo %SFDX_AUTH_URL% > auth.txt
                    sfdx force:auth:sfdxurl:store -f auth.txt -a DevHub
                """
            }
        }

        stage('Run Apex Tests') {
            steps {
                bat """
                    sfdx force:apex:test:run --resultformat json --codecoverage --outputdir test-results --wait 10
                """
            }
            post {
                always {
                    junit 'test-results/test-result-*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        bat """
                            sonar-scanner ^
                              -Dsonar.host.url=http://localhost:9000 ^
                              -Dsonar.login=%SONAR_TOKEN% ^
                              -Dsonar.javascript.lcov.reportPaths=test-results/lcov.info ^
                              -Dsonar.apex.coverage.reportPaths=test-results/test-result-codecoverage.json
                        """
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
