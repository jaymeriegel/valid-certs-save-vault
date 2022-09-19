pipeline {
    agent any
    parameters {
      text name: 'DOMAIN', defaultValue: 'dimed.com.br', description: 'Domain to update certs'
    }
    stages {
        stage('Get CA-Bundle and Private-Key by Vault') {
            agent {
                 docker {
                    image 'jaymeriegel/vault-openssl-jq:v1'
                }
            }
            steps {
                withCredentials([[$class: 'VaultTokenCredentialBinding', addrVariable: 'VAULT_ADDR', credentialsId: 'vault-jenkins-role', tokenVariable: 'VAULT_TOKEN', vaultAddr: 'http://172.17.0.1:8201']]) {
                    script {
                        PRIVATE_KEY = sh (
                            script: 'vault read -field=private_key secrets/creds/certificate_ca/${DOMAIN}',
                            returnStdout: true)
                        CA_BUNDLE = sh (
                            script: 'vault read -field=ca_bundle secrets/creds/certificate_ca/${DOMAIN}',
                            returnStdout: true)
                        
                        writeFile file: 'ca', text: CA_BUNDLE
                        writeFile file: 'key', text: PRIVATE_KEY
                    }
                }
            }
        }
        stage('Provision') {
            steps {
                script {
                        sh """
                        cat ca
                        ansible --version
                        """
                }
            }
        }
    }
}