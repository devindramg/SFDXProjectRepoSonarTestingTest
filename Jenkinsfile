pipeline {
    agent any

    tools {
        // Only include if you also use Maven/JDK
        maven 'Maven3'
        jdk 'Java11'
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/devindramg/SFDXProjectRepoSonarTestingTest.git',
                    credentialsId: 'my-github-token'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    // Use the SonarScanner you configured
                    tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    bat """
                        SonarScanner ^
                          -Dsonar.projectKey=MyProject ^
                          -Dsonar.sources=src ^
                          -Dsonar.host.url=http://localhost:9000 ^
                          -Dsonar.login=sonar-token
                    """
                }
            }
        }
    }
}
