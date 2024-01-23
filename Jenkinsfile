pipeline {
    agent any
    parameters {
        choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'What action should Terraform take?')
    }
    stages {
        stage('Terraform') {
            steps {
                script {
                        dir('Terraform') {
                            sh 'terraform init'
                            sh 'terraform validate'
                            sh "terraform ${params.ACTION} -auto-approve"
                            if (params.ACTION == 'apply') {
                            sh "terraform ${params.ACTION} -auto-approve"
                            // def ip_address = sh(script: 'terraform output public_ip', returnStdout: true).trim()
                            // writeFile file: '../Ansible/inventory', text: "monitoring-server ansible_host=${ip_address}"
                            }
                        }
                }
            }
        }
        stage('Delay') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                script {
                    echo 'Waiting for SSH agent...'
                    sleep 12 // waits for 120 seconds before continuing
                }
            }
        }
        stage('Ansible') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                    dir('Ansible') {
                        sh "ansible-playbook -i inventory appPlaybook.yaml"
                    }
            }
        }
        stage('Output URLs') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                script {
                        dir('./') {
                            sh 'echo "sample url" > urls.txt'
                            // def ip_address = sh(script: 'terraform output public_ip', returnStdout: true).trim()
                            // writeFile file: '../Ansible/inventory', text: "monitoring-server ansible_host=${ip_address}"
                            }
                        }
            }
        }
        stage('Archive URLs') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                archiveArtifacts artifacts: 'urls.txt', onlyIfSuccessful: true
            }
        }
    }
}
