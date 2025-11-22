pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = 'github-token'

        // Inject AWS credentials
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

                rem Step 1: Copy WAR to home directory of EC2 (upload succeeds without sudo)
                "C:\\Program Files\\PuTTY\\pscp.exe" -i "C:\\Users\\samruddhi.bansode\\Downloads\\ipat-eunorth1.ppk" ..\\target\\*.war ubuntu@%EC2_IP%:/home/ubuntu/

                rem Step 2: Move WAR into Tomcat10 webapps folder using sudo INSIDE EC2
                "C:\\Program Files\\PuTTY\\plink.exe" -i "C:\\Users\\samruddhi.bansode\\Downloads\\ipat-eunorth1.ppk" ubuntu@%EC2_IP% \"sudo mv /home/ubuntu/*.war /var/lib/tomcat10/webapps/\"

                rem Step 3: Restart Tomcat10
                "C:\\Program Files\\PuTTY\\plink.exe" -i "C:\\Users\\samruddhi.bansode\\Downloads\\ipat-eunorth1.ppk" ubuntu@%EC2_IP% \"sudo systemctl restart tomcat10\"
                '''
            }
        }
    }
}
