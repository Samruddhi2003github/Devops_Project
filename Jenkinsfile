pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = 'github-token'
    }

    stages {

        stage('Clone Repo') {
            steps {
                git(
                    url: 'https://github.com/Samruddhi2003github/Devops_Project.git',
                    credentialsId: 'github-token',
                    branch: 'main'
                )
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Terraform Init') {
            steps {
                sh '''
                cd terraform
                terraform init
                '''
            }
        }

        stage('Terraform Apply') {
            steps {
                sh '''
                cd terraform
                terraform apply -auto-approve
                '''
            }
        }

        stage('Deploy WAR to Tomcat') {
            steps {
                sh '''
                cd terraform
                EC2_IP=$(terraform output -raw public_ip)

                echo "Deploying to $EC2_IP"

                scp -o StrictHostKeyChecking=no ../target/*.war ubuntu@$EC2_IP:/var/lib/tomcat9/webapps/
                ssh -o StrictHostKeyChecking=no ubuntu@$EC2_IP "sudo systemctl restart tomcat9"
                '''
            }
        }
    }
}
