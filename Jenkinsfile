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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    bat """
                        sonar-scanner ^
                          -Dsonar.projectKey=MyProject ^
                          -Dsonar.sources=src ^
                          -Dsonar.host.url=http://localhost:9000 ^
                          -Dsonar.login=YOUR_SONAR_TOKEN
                    """
                }
            }
        }
    }
}
