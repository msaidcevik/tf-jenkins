pipeline {
    agent any
    tools {
        terraform 'terraform-01'
    }

    environment {
        USER_NAME = "ubuntu"
    }
    
    stages {
        stage('Github Checkout') {
            steps {
                git credentialsId: 'a5045a26-3ca3-4568-918c-2a68698cdb32', url: 'https://github.com/msaidcevik/tf-jenkins-demo.git'
            }
        }
        
        stage('Terraform Init') {
            steps {
                sh 'terraform init -no-color'
            }
        }
        
        stage('Terraform apply') {
            steps {
                sh 'terraform ${action} -auto-approve -no-color'
            }
        }
        
        stage('Wait the instance') {
            steps {
                script {
                    echo 'Waiting for the instance'
                    id = sh(script: 'aws ec2 describe-instances --filters Name=tag-value,Values=roman-app Name=instance-state-name,Values=running --query Reservations[*].Instances[*].[InstanceId] --output text',  returnStdout:true).trim()
                    sh 'aws ec2 wait instance-status-ok --instance-ids $id'
                }
            }
        }
        
        stage('Connect Instance') {
            steps {
                echo 'Connect the created instance'
                script {
                    env.SERVER_IP = sh(script: 'terraform output -raw server_public_ip', returnStdout:true).trim()
                }
                sh 'echo ${SERVER_IP}'
                
                sshagent(['firstkey']) {
                    sh 'ssh -o StrictHostKeyChecking=no -l ubuntu ${SERVER_IP} mkdir smoky123'
                } 
            }
        }
    }
}