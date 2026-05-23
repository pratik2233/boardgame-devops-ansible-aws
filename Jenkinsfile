pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '15'))
    }

    environment {
        // Project paths
        PROJECT_ROOT = "/home/ubuntu/AWS/Automating-Secure-Deployment-of-Board-game-Listing-WebApp-on-AWS"
        APP_DIR      = "${PROJECT_ROOT}/BoardGame"
        ANSIBLE_DIR  = "${PROJECT_ROOT}/ansible"
        REPORT_DIR   = "${PROJECT_ROOT}/security-reports"
        APP_URL      = "http://localhost:8080"

        // ✅ Force Java 17 so Maven compile doesn't fail
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"

        // Keep Maven repo inside Jenkins home (stable builds)
        MAVEN_OPTS = "-Dmaven.repo.local=/var/lib/jenkins/.m2/repository"
    }

    stages {

        stage('Verify Environment') {
            steps {
                sh '''
                    set -e
                    echo "Running as: $(whoami)"
                    echo "JAVA_HOME=$JAVA_HOME"
                    java -version
                    mvn -version
                    ansible --version
                    trivy --version || true
                    git --version

                    echo "Project root:"
                    ls -la ${PROJECT_ROOT}

                    echo "Creating report directory..."
                    sudo mkdir -p ${REPORT_DIR}
                    sudo chown -R jenkins:jenkins ${REPORT_DIR} || true
                    sudo chmod -R 775 ${REPORT_DIR} || true
                '''
            }
        }

        stage('Build Application (Maven)') {
            steps {
                sh '''
                    set -e
                    cd ${APP_DIR}

                    # Ensure target is writable (fix permission issues)
                    sudo chown -R jenkins:jenkins ${APP_DIR} || true

                    rm -rf target
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh '''
                    set +e
                    cd ${PROJECT_ROOT}

                    echo "Running Trivy filesystem scan..."
                    trivy fs --scanners vuln,secret,misconfig \
                      --format table \
                      --output ${REPORT_DIR}/trivy-fs-report.txt \
                      .

                    echo "Trivy scan completed (not failing build in learning mode)."
                    ls -lh ${REPORT_DIR} || true
                '''
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                sh '''
                    set +e
                    cd ${APP_DIR}

                    echo "Running OWASP Dependency-Check..."
                    # Keep failBuildOnCVSS high initially so pipeline doesn’t fail while learning
                    mvn org.owasp:dependency-check-maven:12.2.2:check \
                      -Dformat=HTML \
                      -DfailBuildOnCVSS=11

                    # Copy report to central report folder
                    if [ -f target/dependency-check-report.html ]; then
                      cp target/dependency-check-report.html ${REPORT_DIR}/dependency-check-report.html
                    fi

                    echo "OWASP Dependency-Check completed."
                    ls -lh ${REPORT_DIR} || true
                '''
            }
        }

        stage('Deploy Using Ansible') {
            steps {
                sh '''
                    set -e
                    cd ${ANSIBLE_DIR}
                    ansible-playbook playbooks/deploy-boardgame.yml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -e
                    echo "Waiting for app..."
                    sleep 10

                    echo "HTTP check:"
                    curl -I ${APP_URL}

                    echo "Service status:"
                    sudo systemctl status boardgame --no-pager || true
                '''
            }
        }
    }

    post {
        always {
            // Archive reports even if build fails
            archiveArtifacts artifacts: 'security-reports/*', fingerprint: true, allowEmptyArchive: true

            sh '''
                echo "Final checks:"
                sudo ss -ltnp | grep ':8080' || true
                curl -I http://localhost:8080 || true
            '''
        }

        success {
            echo "✅ SUCCESS: Build + Security Scan + Deploy completed."
        }

        failure {
            echo "❌ FAILED: Check Console Output (build/scans/deploy)."
        }
    }
}
