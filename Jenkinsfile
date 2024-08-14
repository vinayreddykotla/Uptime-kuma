pipeline{
    agent any
    tools {
        jdk 'jdk22'
        nodejs 'node22'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage ("Git Pull"){
            steps{
                git branch: 'main', url: 'https://github.com/vinayreddykotla/Uptime-kuma.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Chatbot \
                    -Dsonar.projectKey=Chatbot '''
                }
            }
        }
        stage('sonar-quality-gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.json"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t uptime ."
                       sh "docker tag uptime vinayreddykotla/uptime:latest "
                       sh "docker push vinayreddykotla/uptime:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image vinayreddykotla/uptime:latest > trivy.json" 
            }
        }
        stage ("Remove container") {
            steps{
                sh "docker stop uptime | true"
                sh "docker rm uptime | true"
             }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name uptime -v /var/run/docker.sock:/var/run/docker.sock -p 3001:3001 vinayreddykotla/uptime:latest'
            }
        }
    }
}
