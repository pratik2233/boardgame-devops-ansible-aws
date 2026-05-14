pipeline {
    agent any

    environment {
        PROJECT_ROOT = "/home/ubuntu/AWS/Automating-Secure-Deployment-of-Board-game-Listing-WebApp-on-AWS"
        APP_DIR = "/home/ubuntu/AWS/Automating-Secure-Deployment-of-Board-game-Listing-WebApp-on-AWS/BoardGame"
        ANSIBLE_DIR = "/home/ubuntu/AWS/Automating-Secure-Deployment-of-Board-game-Listing-WebApp-on-AWS/ansible"
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "/usr/lib/jvm/java-17-openjdk-amd64/bin:${env.PATH}"
    }

    stages {
        stage('Workspace Info') {
            steps {
                echo 'Starting BoardGame CI/CD Pipeline'
                sh 'whoami'
                sh 'pwd'
                sh 'java -version'
                sh 'mvn -version'
                sh 'docker --version'
                sh 'ansible --version'
            }
        }

        stage('Maven Build') {
            steps {
                dir("${APP_DIR}") {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir("${APP_DIR}") {
                    sh 'docker build -t boardgame:1.0 .'
                }
            }
        }

        stage('Deploy Using Ansible') {
            steps {
                dir("${ANSIBLE_DIR}") {
                    sh 'ansible-playbook playbooks/deploy-boardgame.yml'
                }
            }
        }

        stage('Health Check') {
            steps {
                sh 'sleep 10'
                sh 'curl -I http://localhost:8081'
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: BoardGame application deployed successfully using Jenkins and Ansible.'
        }

        failure {
            echo 'FAILED: BoardGame CI/CD pipeline failed. Please check console output.'
        }
    }
}
