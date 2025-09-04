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
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        bat """
                            sonar-scanner ^
                              -Dsonar.projectKey=MyProject ^
                              -Dsonar.sources=src ^
                              -Dsonar.host.url=http://localhost:9000 ^
                              -Dsonar.login=%SONAR_TOKEN%
                        """
                    }
                }
            }
        }
    }
}
