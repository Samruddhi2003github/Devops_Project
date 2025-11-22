pipeline {
    agent any

    environment {
        AWS_REGION = "eu-north-1"

        // These are already created in your Jenkins
        AWS_ACCESS_KEY_ID     = credentials('aws_access_key')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
    }

    stages {

        stage('Checkout Code') {
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
                bat "mvn clean package"
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
                    def rawIp = bat(
                        script: '@echo off & terraform -chdir=terraform output -raw public_ip',
                        returnStdout: true
                    ).trim()

                    env.EC2_IP = rawIp
                    echo "EC2 Public IP = ${env.EC2_IP}"
                }
            }
        }

        stage("Wait for EC2 Instance") {
            steps {
                echo "Waiting 30 seconds for EC2 to be ready..."
                sleep(time: 30, unit: "SECONDS")
            }
        }

        stage('Deploy WAR to Tomcat10') {
            steps {
                script {

                    def PPK = "C:\\\\ProgramData\\\\Jenkins\\\\keys\\\\ipat-eunorth1.ppk"

                    echo "Uploading WAR to EC2: ${env.EC2_IP}"

                    // Step 1: Upload WAR file
                    bat """
"C:\\Program Files\\PuTTY\\pscp.exe" -i "${PPK}" target\\firststsproject-0.0.1-SNAPSHOT.war ubuntu@${EC2_IP}:/home/ubuntu/app.war
"""

                    // Step 2: Move WAR into Tomcat10
                    bat """
"C:\\Program Files\\PuTTY\\plink.exe" -i "${PPK}" ubuntu@${EC2_IP} "sudo mv /home/ubuntu/app.war /var/lib/tomcat10/webapps/ && sudo systemctl restart tomcat10"
"""

                    echo "Deployment Completed Successfully!"
                }
            }
        }
    }
}
