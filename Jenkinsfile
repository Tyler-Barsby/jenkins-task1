pipeline {
    agent { label 'worker' }

    stages {

        stage('Init') {
            steps {
                sh 'docker rm -f flask-app nginx || true'
                sh 'docker network rm app-network || true'
                sh 'docker network create app-network'
                echo 'Init done'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t flask-app .'
                echo 'Build done'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL -f json -o trivy-results.json .'
                sh 'trivy fs --severity HIGH,CRITICAL .'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-results.json'
                }
            }
        }

        stage('Approve') {
            steps {
                input message: 'Review the Trivy scan results. Approve to continue deployment?', ok: 'Deploy'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker run -d --name flask-app --network app-network flask-app'
                sh 'docker run -d -p 80:80 --name nginx --network app-network --mount type=bind,source=$(pwd)/nginx.conf,target=/etc/nginx/nginx.conf nginx'
                echo 'Deploy done'
            }
        }

    }

    post {
        success {
            echo 'it worked!'
        }
        failure {
            echo 'something went wrong'
            sh 'docker ps -a'
        }
    }
}
