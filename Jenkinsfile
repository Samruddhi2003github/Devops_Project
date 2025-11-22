pipeline {
    agent any

    environment {
        AWS_REGION = "eu-north-1"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/rohitswabhav/firststsproject.git'
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

        stage("Get EC2 IP") {
            steps {
                script {
                    def rawIp = bat(
                        script: '@echo off & terraform -chdir=terraform output -raw public_ip',
                        returnStdout: true
                    ).trim()

                    env.EC2_IP = rawIp.split()[-1].trim()
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

        stage("Install Tomcat & Deploy WAR") {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ipat-eunorth1', keyFileVariable: 'KEYFILE', usernameVariable: 'SSH_USER')]) {
                    script {

                        echo "Deploying WAR file to EC2..."

                        // Upload WAR
                        bat """
for %%f in (target\\*.war) do (
  echo Uploading WAR: %%f
  pscp -v -batch -i "%KEYFILE%" -hostkey "ssh-ed25519 255 SHA256:pWWe3ooQ+CaZneciIemS50Cl9vFT75LL3g404xy08kg" "%%f" %SSH_USER%@${EC2_IP}:/home/%SSH_USER%/app.war
)
"""

                        // Verify
                        bat """
plink -batch -i "%KEYFILE%" -hostkey "ssh-ed25519 255 SHA256:pWWe3ooQ+CaZneciIemS50Cl9vFT75LL3g404xy08kg" ubuntu@${EC2_IP} "ls -lh /home/ubuntu/app.war || echo 'WAR file not found!'"
"""

                        // Move & restart Tomcat10
                        bat """
plink -batch -i "%KEYFILE%" -hostkey "ssh-ed25519 255 SHA256:pWWe3ooQ+CaZneciIemS50Cl9vFT75LL3g404xy08kg" ubuntu@${EC2_IP} "sudo mv /home/ubuntu/app.war /var/lib/tomcat10/webapps/ && sudo systemctl restart tomcat10"
"""
                    }
                }
            }
        }
    }
}
