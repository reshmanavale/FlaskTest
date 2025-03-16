pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/flask_app"  // Directory on EC2 where app will be deployed
        SSH_KEY = "/var/jenkins_home/.ssh/id_rsa"
        EC2_USER = "ubuntu"
        EC2_IP = "13.57.48.63"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: '2e869095-a76c-4b00-85e0-cb0fddb3a460',
                    url: 'git@github.com:reshmanavale/FlaskTest.git'
            }
        }

        stage('Build') {
            steps {
             sh '''
                apt update && apt install -y python3-venv
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                 sh '''
                . venv/bin/activate
                pip list | grep pytest  # Debugging step
                pytest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sh 'scp -o StrictHostKeyChecking=no -i /var/jenkins_home/.ssh/id_rsa -r * ubuntu@13.57.48.63:/home/ubuntu/app'
                }
            }
        }
    }
}

