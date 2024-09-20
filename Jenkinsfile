pipeline{
     agent any
     
     tools{
         jdk 'jdk17'
         nodejs 'node16'
     }
     environment {
         SCANNER_HOME=tool 'sonarqube-scanner'
     }
     
     stages {
         stage('Clean Workspace'){
             steps{
                 cleanWs()
             }
         }
         stage('Checkout from Git'){
             steps{
                 git branch: 'main', url: 'https://github.com/mrbalraj007/a-swiggy-clone'
             }
         }
        stage("Sonarqube Analysis "){
             steps{
                 withSonarQubeEnv('SonarQube-Server') {
                     sh ''' 
                     $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy-CICD \
                     -Dsonar.projectKey=Swiggy-CICD 
                     '''
                 }
             }
         }
        stage("Quality Gate"){
            steps {
                 script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token' 
                 }
             } 
         }
        stage('Install Dependencies') {
             steps {
                 sh "npm install"
             }
         }
         stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
         }
        stage("Docker Build & Push"){
             steps{
                 script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
                        sh "docker build -t swiggy-clone ."
                        sh "docker tag swiggy-clone balrajsi/swiggy-clone:latest "
                        sh "docker push balrajsi/swiggy-clone:latest "
                     }
                 }
             }
         }
        stage("TRIVY"){
             steps{
                 sh "trivy image balrajsi/swiggy-clone:latest > trivyimage.txt" 
             }
         }
        
        stage('Deploy to Kubernets'){
             steps{
                 script{
                     dir('Kubernetes') {
                         withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                         sh 'kubectl delete --all pods'
                         sh 'kubectl apply -f deployment.yml'
                         sh 'kubectl apply -f service.yml'
                         }   
                     }
                 }
             }
         } 
    }
}
