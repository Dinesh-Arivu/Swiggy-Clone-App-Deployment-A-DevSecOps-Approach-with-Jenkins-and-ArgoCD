pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        GIT_REPO_NAME = "Swiggy-Clone-App-Deployment-A-DevSecOps-Approach-with-Jenkins-and-ArgoCD.git"
        GIT_USER_NAME = "Dinesh-Arivu"     
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Dinesh-Arivu/Swiggy-Clone-App-Deployment-A-DevSecOps-Approach-with-Jenkins-and-ArgoCD.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=swiggy \
                    -Dsonar.projectKey=swiggy '''
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
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t swiggy ."
                       sh "docker tag swiggy dinesh1097/swiggy:latest "
                       sh "docker push dinesh1097/swiggy:latest "
                    }
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image dinesh1097/swiggy:latest > trivyimage.txt"
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Dinesh-Arivu/Swiggy-Clone-App-Deployment-A-DevSecOps-Approach-with-Jenkins-and-ArgoCD.git'
            }
        }
        stage('Update Deployment File') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                       NEW_IMAGE_NAME = "dinesh1097/swiggy:latest"   
                       sh "sed -i 's|image: .*|image: $NEW_IMAGE_NAME|' K8S-manifest/deployment.yml"
                       sh 'git add K8S-manifest/deployment.yml'
                       sh "git commit -m 'Update deployment image to $NEW_IMAGE_NAME'"
                       sh "git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main"
                    }
                }
            }
        }
    }
}
