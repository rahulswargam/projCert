pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'swargamrahul/edureka'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'git@github.com:rahulswargam/projCert.git', credentialsId: 'ssh-key'
            }
        }

        stage('Provision Test Server') {
            steps {
                sh 'ansible-playbook -i /etc/ansible/hosts setup-test-server.yml'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([
                    string(credentialsId: 'Docker Username', variable: 'DOCKER_USER'),
                    string(credentialsId: 'Docker Password', variable: 'DOCKER_PASS')
                ]) {
                    sh '''
                    docker build -t $DOCKER_HUB_REPO:latest .
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_HUB_REPO:latest
                    '''
                }
            }
        }

        stage('Deploy to Test Server') {
            steps {
                sh 'ansible-playbook -i /etc/ansible/hosts deploy-test.yml'
            }
        }

        stage('Approval for Production Deployment') {
            steps {
                input message: 'Deploy to production?', ok: 'Proceed'
            }
        }

        stage('Deploy to Production') {
            steps {
                sh 'ansible-playbook -i /etc/ansible/hosts deploy-prod.yml'
            }
        }
    }
}
