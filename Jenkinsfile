pipeline {
    agent any
    tools{
       jdk 'java17'
       nodejs 'node16'
    }
    environment {
        SCANNER_HOME =tool 'sonar-scanner'
    }
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sanjaybaskar/Tetris-CICD-v2.git'
            }
        }
        stage('Sonar analysis') {
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=TetrisV2 \
                    -Dsonar.projectKey=Tetris '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
            }
        }
        stage('NPM Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Trivy scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
          stage('Owasp DP Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
         stage('Docker Image ') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker',toolName: 'docker') {
                            sh '''
                            docker build -t tetrisv2 .
                            docker tag tetrisv2 sanjaybaskar/tetrisv2:latest
                            docker push sanjaybaskar/tetrisv2:latest
                            '''
                            }
                }
            }
        }
        stage('Trivy Image scan') {
            steps {
                sh 'trivy image sanjaybaskar/tetrisv2:latest > trivyimage.txt'
            }
        }
        stage('Trigger manifest Pipeline'){
            steps{
                build job:'Manifest-job', wait:true
            }
        }
    }
}
