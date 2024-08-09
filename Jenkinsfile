pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node19'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'legacy', url: 'https://github.com/sagark917/chatbot-ui.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        #stage("Sonarqube Analysis "){
        #    steps{
         #       withSonarQubeEnv('sonar-server') {
         #           sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Chatbot \
         #           -Dsonar.projectKey=Chatbot '''
          #      }
        #  }
        #}
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        #stage('OWASP FS SCAN') {
         #   steps {
         #       dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
         #       dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
          #  }
        #}
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.json"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t chatbot ."
                       sh "docker tag chatbot sagaramd/chatbot:latest "
                       sh "docker push sagaramd/chatbot:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image sagaramd/chatbot:latest > trivy.json" 
            }
        }
        stage ("Remove container") {
            steps{
                sh "docker stop chatbot | true"
                sh "docker rm chatbot | true"
             }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name chatbot -p 3000:3000 sagaramd/chatbot:latest'
                }
            }
        stage('Deploy to kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f k8s/chatbot-ui.yaml --validate=false'
                  }
                }
            }
        }
        stage('scan cluster'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'trivy k8s --report summary cluster'
                  }
                }
            }
        }
        }
    }
