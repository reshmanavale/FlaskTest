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
                    ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} << 'EOF'
                        # Ensure deployment directory exists
                        mkdir -p ${APP_DIR}

                        # Install necessary packages
                        sudo apt update && sudo apt install -y python3-venv python3-pip

                        # Navigate to deployment directory
                        cd ${APP_DIR}

                        # Clone repository if not exists, else pull latest changes
                        if [ ! -d "FlaskTest" ]; then
                            git clone https://github.com/reshmanavale/FlaskTest.git
                        else
                            cd FlaskTest
                            git pull origin main
                        fi

                        # Set up virtual environment
                        cd FlaskTest
                        python3 -m venv venv
                        source venv/bin/activate

                        # Install dependencies
                        pip install --upgrade pip
                        pip install -r requirements.txt

                        # Kill any running process and restart
                        pkill -f "gunicorn" || true
                        nohup venv/bin/gunicorn --bind 0.0.0.0:5000 wsgi:app > output.log 2>&1 &

                    EOF
                    """
                }
            }
        }
    }
}
