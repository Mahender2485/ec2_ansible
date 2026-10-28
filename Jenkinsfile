pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_REGION = 'us-east-1'
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout GitHub') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

                    python3 -m venv .venv

                    .venv/bin/python -m pip install --upgrade pip

                    .venv/bin/python -m pip install \
                        ansible-core boto3 botocore

                    .venv/bin/ansible-galaxy collection install \
                        -r requirements.yml

                    .venv/bin/ansible --version
                '''
            }
        }

        stage('Validate Ansible') {
            steps {
                sh '''
                    .venv/bin/ansible-playbook \
                        --syntax-check create-ec2.yml
                '''
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Approve AWS EC2 Creation?',
                          ok: 'Create EC2'
                }
            }
        }

        stage('Create EC2 Instance') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'aws-secret-access-key',
                        variable: 'AWS_ACCESS_KEY_ID'
                    ),
                    string(
                        credentialsId: 'aws-secret-access-key',
                        variable: 'AWS_ACCESS_KEY_ID'
                    )
                ]) {
                    sh '''
                        set -e

                        .venv/bin/ansible-playbook \
                            -i inventory.ini \
                            -e ansible_python_interpreter="$PWD/.venv/bin/python" \
                            create-ec2.yml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'AWS EC2 deployment successful!'
        }

        failure {
            echo 'AWS EC2 deployment failed!'
        }

        aborted {
            echo 'Deployment cancelled.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}