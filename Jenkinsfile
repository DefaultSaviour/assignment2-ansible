pipeline {
    agent any

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'latest',
            description: 'Docker image tag to deploy (e.g. v1.0.5)'
        )
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_SESSION_TOKEN     = credentials('aws-session-token')
        VAULT_PASSWORD_FILE   = credentials('ansible-vault-password')
    }

    stages {

        stage('Get Code') {
            steps {
                echo "Checking out infrastructure repo..."
                checkout scm
            }
        }

        stage('Provision Key and Security Group') {
            steps {
                echo "Provisioning SSH key and security group..."
                sh """
                    ansible-playbook provision_key_security_group.yml \
                        --vault-password-file ${VAULT_PASSWORD_FILE}
                """
            }
        }

        stage('Provision EC2 and Deploy App') {
            steps {
                echo "Provisioning EC2 instance and deploying image: ${params.IMAGE_TAG}..."
                sh """
                    ansible-playbook provision_and_deploy.yml \
                        --vault-password-file ${VAULT_PASSWORD_FILE} \
                        --extra-vars "image_tag=${params.IMAGE_TAG}"
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Verifying application is live at http://34.232.104.121 ..."
                sh "curl -f http://34.232.104.121"
                echo "Deployment verified successfully!"
            }
        }

    }

    post {
        success {
            echo "CD complete. App deployed: defaultsaviour/runcalc-pro:${params.IMAGE_TAG}"
            echo "Live at: http://34.232.104.121"
        }
        failure {
            echo "CD pipeline failed — check the logs above."
        }
    }
}
