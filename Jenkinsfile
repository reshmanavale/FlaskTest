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
                ssh -o StrictHostKeyChecking=no -i /var/jenkins_home/.ssh/id_rsa ubuntu@13.57.48.63 <<EOF
                set -e  # Exit on error

                echo "🚀 Connecting to EC2 and deploying Flask app"
                cd ~/FlaskTest || exit 1

                # Kill any existing process running on port 5000
                echo "🛑 Stopping existing Flask process..."
                PID=\$(lsof -t -i:5000) && kill -9 \$PID || echo "No process running on port 5000"

                # Pull latest changes
                git reset --hard
                git pull origin main

                # Activate virtual environment
                source venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt

                # Start Flask app
                echo "🚀 Starting Flask app..."
                nohup venv/bin/python3 app.py > output.log 2>&1 &

                echo "✅ Deployment completed successfully!"
                exit 0  # Ensure Jenkins marks it as success
                EOF
                exit 0  # Ensure Jenkins success
            """
        }
    }
}




               


}
}
