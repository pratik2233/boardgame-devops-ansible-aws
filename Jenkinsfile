pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        PROJECT_ROOT = "/home/ubuntu/AWS/Automating-Secure-Deployment-of-Board-game-Listing-WebApp-on-AWS"
        ANSIBLE_DIR  = "${PROJECT_ROOT}/ansible"
        APP_DIR      = "${PROJECT_ROOT}/BoardGame"
        APP_URL      = "http://localhost:8080"
    }

    stages {
        stage('Verify Workspace') {
            steps {
                sh '''
                    echo "Running as:"
                    whoami
                    echo "Project root:"
                    ls -la ${PROJECT_ROOT}
                    echo "BoardGame directory:"
                    ls -la ${APP_DIR}
                    echo "Ansible directory:"
                    ls -la ${ANSIBLE_DIR}
                '''
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    java -version
                    mvn -version
                    ansible --version
                    git --version
                '''
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    cd ${APP_DIR}
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Deploy Using Ansible') {
            steps {
                sh '''
                    cd ${ANSIBLE_DIR}
                    ansible-playbook playbooks/deploy-boardgame.yml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 10
                    curl -I ${APP_URL}
                    sudo systemctl status boardgame --no-pager || true
                '''
            }
        }
    }

    post {
        success {
            echo "SUCCESS: BoardGame application deployed successfully through Jenkins CI/CD."
        }

        failure {
            echo "FAILED: Jenkins pipeline failed. Please check the console output."
        }

        always {
            sh '''
                sudo systemctl status boardgame --no-pager || true
                sudo ss -ltnp | grep ':8080' || true
            '''
        }
    }
}
