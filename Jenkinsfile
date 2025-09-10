pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/dushyantkumark/Swiggy_DevSecOps_Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy-CICD \
                    -Dsonar.projectKey=Swiggy-CICD '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
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
                    app.push("${env.BUILD_NUMBER}")  
                       sh "docker build -t swiggy-app ."
                       sh "docker tag swiggy-app dushyantkumark/swiggy-app:latest "
                       sh "docker push dushyantkumark/swiggy-app:latest "
                    }
                }
            }
        }
        stage("TRIVY IMAGE SCAN"){
            steps{
                sh "trivy image dushyantkumark/swiggy-app:latest > trivyimage.txt" 
            }
        }
        stage("Depoy to container"){
            steps{
                sh "docker run -d -p 3000:80 --name swiggy dushyantkumark/swiggy:latest" 
            }
        }
        stage('Deploy to k8s'){
            steps{
                dir('k8s-manifest') {
                  withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'swiggy-cluster', contextName: '', credentialsId: 'k8s', namespace: 'swiggy', serverUrl: '']]) {
                    sh 'kubectl apply -f deployment.yaml'    
                    sh 'kubectl apply -f service.yaml'
                   }
                }   
            }
        }
    }
}
