pipeline {
    agent any
    tools{
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git url: 'https://github.com/thirupathi8790/task-master-pro.git' , branch: 'main'
            }
        }
        
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('unit test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('trivy-fs-scann'){
            steps{
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        
        stage('sonar analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=taskemaster -Dsonar.projectkey=taskemaster\
                    -Dsonar.java.binaries=target '''
                }
            }
        }
        
        stage('build the application'){
            steps{
                sh 'mvn package'
            }
        }
        
        stage('publish artifactor'){
            steps{
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                     sh 'mvn deploy'
                 }
            }
        }
        
         stage('docker build & tag'){
            steps{
                script{
                withDockerRegistry(credentialsId: 'DockerHub', toolName: 'docker') {
                    sh 'docker build -t thirupathinaik/taskmaster:latest .'
                }
            }
         }}
        
         stage('trivy-image-scann'){
            steps{
                sh 'trivy image --format table -o image.html thirupathinaik/taskmaster:latest'
            }
         }
         
          stage('docker push'){
            steps{
                script{
                withDockerRegistry(credentialsId: 'DockerHub', toolName: 'docker') {
                    sh 'docker push thirupathinaik/taskmaster:latest'
                    
                }
            }
        }}
        
        stage('k8s deploy'){
            steps{
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'Eks-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', serverUrl: 'https://C8A9E02883D982EB1C5357FCF9810B02.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh 'kubectl apply -f deployment-service.yml'
                    sleep 30
                    }
            }
        }
        
                stage('k8s deployment'){
            steps{
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'Eks-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', serverUrl: 'https://C8A9E02883D982EB1C5357FCF9810B02.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                    }
            }
        }
    }
}
