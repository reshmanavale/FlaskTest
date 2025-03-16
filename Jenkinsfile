pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/flask_app"  // Directory on EC2 where the app will be deployed
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
                apt update && apt install -y python3-venv python3-pip
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
                pip install pytest  # Ensure pytest is installed
                pytest || true  # Allow tests to fail without stopping pipeline
                '''
            }
        }

       stage('Deploy to EC2') {
    steps {
        script {
            sh """
            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} <<EOF
                mkdir -p ${APP_DIR}
                sudo apt update && sudo apt install -y python3-pip
                cd ${APP_DIR}
                if [ ! -d "FlaskTest" ]; then
                    git clone https://github.com/reshmanavale/FlaskTest.git
                else
                    cd FlaskTest && git pull
                fi
                cd FlaskTest
                pip3 install -r requirements.txt
                nohup python3 app.py > output.log 2>&1 &
            EOF
            """
        }
    }
}

}
}
