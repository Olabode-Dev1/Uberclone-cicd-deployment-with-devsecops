pipeline {
    agent any
    stages {
        stage('Run Terrascan with Docker') {
            steps {
                sh '''
                docker run --rm -v $(pwd):/iac accurics/terrascan:latest scan -d uber-clone/EKS_Terraform
                '''
            }
        }
    }
}
