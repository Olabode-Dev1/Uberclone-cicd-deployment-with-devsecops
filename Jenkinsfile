pipeline {
    agent any
    stages {
        stage('Run Terrascan') {
            steps {
                script {
                    // Run Terrascan and save the JSON output
                    def scanStatus = sh(
                        script: '''
                        docker run --rm -v /var/lib/jenkins/workspace/checkov_terraform:/iac accurics/terrascan:latest scan -d /iac/EKS_Terraform -o json > terrascan_output.json
                        ''',
                        returnStatus: true
                    )
                    
                    // Archive Terrascan results
                    archiveArtifacts artifacts: 'terrascan_output.json', allowEmptyArchive: true
                    
                    // Parse JSON output
                    def jsonContent = readFile('terrascan_output.json')
                    def parsedJSON = new groovy.json.JsonSlurper().parseText(jsonContent)
                    
                    // Count medium and high severity violations
                    def mediumViolations = parsedJSON.results.violations.findAll { it.severity == 'MEDIUM' }.size()
                    def highViolations = parsedJSON.results.violations.findAll { it.severity == 'HIGH' }.size()

                    // Fail pipeline if medium or high severity vulnerabilities exist
                    if (mediumViolations > 0 || highViolations > 0) {
                        error("Terrascan found ${mediumViolations} medium and ${highViolations} high severity vulnerabilities. Check terrascan_output.json for details.")
                    } else {
                        echo "No critical vulnerabilities found. Proceeding with Terraform commands."
                    }
                }
            }
        }
        stage('Terraform Init') {
            steps {
                sh '''
                cd /var/lib/jenkins/workspace/checkov_terraform/EKS_Terraform
                terraform init
                '''
            }
        }
        stage('Terraform Validate') {
            steps {
                sh '''
                cd /var/lib/jenkins/workspace/checkov_terraform/EKS_Terraform
                terraform validate
                '''
            }
        }
        stage('Terraform Apply') {
            steps {
                // Automatically approve the apply step
                sh '''
                cd /var/lib/jenkins/workspace/checkov_terraform/EKS_Terraform
                terraform ${action} -auto-approve
                '''
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
        failure {
            echo 'Pipeline failed. Please check the logs for details.'
        }
    }
}
