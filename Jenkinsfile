pipeline {
    agent any

    environment {
        VIRTUAL_ENV = 'venv'
        PYTHON = 'python3'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/reshmanavale/FlaskTest.git'
            }
        }

        stage('Setup Python') {
            steps {
                sh 'python3 -m venv $VIRTUAL_ENV'
                sh 'source $VIRTUAL_ENV/bin/activate'
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'source $VIRTUAL_ENV/bin/activate && pytest'
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'scp -r ./* ubuntu@your-ec2-instance:/var/www/flask-app'
                sh 'ssh ubuntu@your-ec2-instance "sudo systemctl restart flask-app"'
            }
        }
    }

    post {
        success {
            mail to: 'your-email@example.com',
                 subject: "Jenkins Build Successful: ${env.BUILD_NUMBER}",
                 body: "The pipeline has completed successfully."
        }
        failure {
            mail to: 'your-email@example.com',
                 subject: "Jenkins Build Failed: ${env.BUILD_NUMBER}",
                 body: "Please check the build logs."
        }
    }
}
