pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
    environment {
        registry = '730137084652.dkr.ecr.us-east-1.amazonaws.com/geolocation_ecr_rep'
	registryCredential = 'jenkins-ecr'
        dockerimage = '' 
    }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/tabenjamin2000/jenkins-ECR-EKS.git'
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        // Building Docker images
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry
                }
            }
        }
        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
		    docker.withRegistry("https://"+registry,"ecr:us-east-1:"+registryCredential) {
                        dockerImage.push()
                }
            }
        }
        //deploy the image that is in ECR to our EKS cluster
        stage ("Kube Deploy") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'eks_credential', namespace: '', serverUrl: '') {
                 sh "kubectl apply -f eks-deploy-from-ecr.yaml"
                }
            }
        }
    }
}
