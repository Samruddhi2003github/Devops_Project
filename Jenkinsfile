pipeline {
    agent any

    environment {
        AWS_REGION = "eu-north-1"
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
                withCredentials([
                    string(credentialsId: 'aws_access_key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws_secret_key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    bat 'terraform -chdir=terraform apply -auto-approve'
                }
            }
        }

        stage("Get EC2 IP") {
            steps {
                script {
                    def output = bat(
                        script: '@echo off & terraform -chdir=terraform output -raw public_ip',
                        returnStdout: true
                    ).trim()

                    env.EC2_IP = output
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

        stage("Deploy WAR to Tomcat10") {
            steps {
                script {

                    echo "Uploading WAR to EC2: ${env.EC2_IP}"

                    // Upload WAR file
                    bat """
                    pscp -i "C:\\Users\\samruddhi.bansode\\Downloads\\ipat-eunorth1.ppk" target\\*.war ubuntu@${EC2_IP}:/home/ubuntu/app.war
                    """

                    // Move to Tomcat10 + restart
                    bat """
                    plink -i "C:\\Users\\samruddhi.bansode\\Downloads\\ipat-eunorth1.ppk" ubuntu@${EC2_IP} "sudo mv /home/ubuntu/app.war /var/lib/tomcat10/webapps/ && sudo systemctl restart tomcat10"
                    """
                }
            }
        }
    }
}
