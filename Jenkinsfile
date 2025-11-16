pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = 'github-token'

        // Inject AWS credentials from Jenkins credentials store
        AWS_ACCESS_KEY_ID     = credentials('aws_access_key')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
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
                bat 'mvn clean package'
            }
        }

        stage('Terraform Init') {
            steps {
                bat '''
                cd terraform
                terraform init
                '''
            }
        }

        stage('Terraform Apply') {
            steps {
                bat '''
                cd terraform
                terraform apply -auto-approve
                '''
            }
        }

        stage('Deploy WAR to Tomcat') {
            steps {
                bat '''
                cd terraform

                rem Fetch public IP from Terraform output
                for /f %%i in ('terraform output -raw public_ip') do set EC2_IP=%%i

                echo Deploying to %EC2_IP%

                rem Copy WAR file to EC2 using PuTTY pscp
                "C:\\Program Files\\PuTTY\\pscp.exe" -i "C:\\Users\\samruddhi.bansode\\Downloads\\ipat-eunorth1.ppk" ..\\target\\*.war ubuntu@%EC2_IP%:/var/lib/tomcat9/webapps/

                rem Restart tomcat via SSH using plink
                "C:\\Program Files\\PuTTY\\plink.exe" -i "C:\\Users\\samruddhi.bansode\\Downloads\\ipat-eunorth1.ppk" ubuntu@%EC2_IP% "sudo systemctl restart tomcat9"
                '''
            }
        }
    }
}
