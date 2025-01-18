properties([
    parameters([
        choice(
            choices: ['dev', 'staging'], 
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'], 
            name: 'Terraform_Action'
        )
    ])
])
pipeline {
    agent any
    stages {
        stage('Preparing') {
            steps {
                sh 'echo Preparing'
            }
        }
        stage('Git Pulling') {
            steps {
                git branch: 'master', url: 'https://github.com/AmanPathak-DevOps/EKS-Terraform-GitHub-Actions.git'
            }
        }
        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'terraform -chdir=eks/ init'
                }
            }
        }
        stage('Validate') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'terraform -chdir=eks/ validate'
                }
            }
        }

        stage('Setup Terraform Workspace') {
            steps {
                script {
                    echo "Setting up Terraform workspace for environment: ${params.Environment}"
                    sh """
                        terraform workspace select ${params.Environment} || terraform workspace new ${params.Environment}
                    """
                }
            }
        }
        
        stage('Action') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {
                        echo "Executing Terraform Action: ${params.Terraform_Action} in the ${params.Environment} workspace"
    
                        // Execute Terraform commands based on the selected action
                        if (params.Terraform_Action == 'plan') {
                            sh "terraform plan -var-file=${params.Environment}.tfvars"
                        } else if (params.Terraform_Action == 'apply') {
                            sh "terraform apply -var-file=${params.Environment}.tfvars -auto-approve"
                        } else if (params.Terraform_Action == 'destroy') {
                            sh "terraform destroy -var-file=${params.Environment}.tfvars -auto-approve"
                        }
                    }
                }
            }
        }
    }
}
