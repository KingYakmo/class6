pipeline {
    agent any
    envioronment {
        AWS_REGION = 'eu-west-1'
    }
    stages {    
        stage("Set AWS Credentials") {
            steps {
                withCredentials([
                    $class: "AmazonWebServicesCredentialsBinding",
                    credentialsId: 'AWS-Jenkins'
                ]) {
                    sh '''
                    echo "AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID"
                    aws sts get-caller-identity
                    '''
                }
        }
        stage('Checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/KingYakmo/class6.git'
            }
        }
        stage('Initialize Terraform') {
            steps {
                sh '''
                terraform init
                '''
            }
        }
        stage('Plan Terraform') {
            steps {
                withCredentials([
                    $class: "AmazonWebServicesCredentialsBinding",
                    credentialsId: 'AWS-Jenkins'
                ]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform plan -out=tfplan
                    '''
            }
        }
        stage('Apply Terraform') {
            steps {
                input message: "Approve Terraform Apply?", ok: "Apply"
                withCredentials([
                    $class: "AmazonWebServicesCredentialsBinding",
                    credentialsId: 'AWS-Jenkins'
                ]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform apply tfplan
                    '''
                }
            }
        }
    stage ('Destroy Terraform') {
        steps {
            input message: "Do you want to destroy the Infrastructure?", ok: "Destroy"
            withCredentials([
                $class: "AmazonWebServicesCredentialsBinding",
                credentialsId: 'AWS-Jenkins'
            ]) {
                sh '''
                export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                terraform destroy -auto-approve
                '''
            }
        }
    }
}
post {
    sucess {
        echo 'Terraform depployment completed successfully!'
    }
    failure {
        echo 'Terraform deployment failed!'
    }
}
}