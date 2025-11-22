pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws_access_key')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
        PATH = "C:\\Program Files\\Terraform;C:\\Program Files\\PuTTY;${env.PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Samruddhi2003github/Devops_Project.git',
                    branch: 'main',
                    credentialsId: 'github-token'
            }
        }

        stage('Maven Build') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Terraform Init') {
            steps {
                bat 'terraform -chdir=terraform init'
            }
        }

        stage('Terraform Apply') {
            steps {
                bat 'terraform -chdir=terraform apply -auto-approve'
            }
        }

        stage('Get EC2 IP') {
            steps {
                script {
                    def output = bat(returnStdout: true, script: '''
                        terraform -chdir=terraform output -raw public_ip
                    ''').trim()

                    env.EC2_PUBLIC_IP = output
                    echo "EC2 Public IP = ${env.EC2_PUBLIC_IP}"
                }
            }
        }

        stage('Wait for EC2') {
            steps {
                echo "Waiting 20 seconds for EC2..."
                sleep 20
            }
        }

        stage('Deploy WAR to Tomcat10') {
            steps {
                script {
                    // IMPORTANT: Disable host key checking (Alok style)
                    echo "Uploading WAR to EC2: ${env.EC2_PUBLIC_IP}"

                    bat """
                    echo y | "C:\\Program Files\\PuTTY\\pscp.exe" ^
                        -i "C:\\ProgramData\\Jenkins\\keys\\ipat-eunorth1.ppk" ^
                        target\\firststsproject-0.0.1-SNAPSHOT.war ^
                        ubuntu@${env.EC2_PUBLIC_IP}:/home/ubuntu/app.war
                    """

                    // Restart Tomcat
                    bat """
                    echo y | "C:\\Program Files\\PuTTY\\plink.exe" ^
                        -i "C:\\ProgramData\\Jenkins\\keys\\ipat-eunorth1.ppk" ^
                        ubuntu@${env.EC2_PUBLIC_IP} "sudo systemctl restart tomcat9"
                    """
                }
            }
        }
    }
}
