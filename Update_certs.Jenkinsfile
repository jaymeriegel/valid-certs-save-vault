pipeline {
    agent any
    parameters {
      text name: 'DOMAIN', defaultValue: 'dimed.com.br', description: 'Domain to update certs'
      text name: 'USER', defaultValue: 'nodo', description: 'user remote server'
    }
    environment {
                VAULT_ADDR = "http://172.17.0.1:8201"
                CREDENTIALS_SSH = "jenkins"
                VAULT_PATH_TO_GET_SECRETS = "secrets/creds/certificate_ca"
    }
    stages {
        stage('Get CA-Bundle and Private-Key by Vault') {
            agent {
                 docker {
                    image 'jaymeriegel/vault-openssl-jq:v1'
                }
            }
            steps {
                withCredentials([[$class: 'VaultTokenCredentialBinding', addrVariable: 'VAULT_ADDR', credentialsId: 'vault-jenkins-role', tokenVariable: 'VAULT_TOKEN', vaultAddr: VAULT_ADDR]]) {
                    script {
                        PRIVATE_KEY = sh (
                            script: 'vault read -field=private_key ${VAULT_PATH_TO_GET_SECRETS}/${DOMAIN}',
                            returnStdout: true)
                        CA_BUNDLE = sh (
                            script: 'vault read -field=ca_bundle ${VAULT_PATH_TO_GET_SECRETS}/${DOMAIN}',
                            returnStdout: true)
                    }
                }
            }
        }
        stage('Provision') {
            steps {
                sshagent (credentials: [CREDENTIALS_SSH]) {
                    script {
                        writeFile file: 'ca', text: CA_BUNDLE
                        writeFile file: 'key', text: PRIVATE_KEY
                        sh """
                        #!/bin/bash
                        ansible ${DOMAIN} -m copy -a "src=ca dest=/home/nodo/certs/${DOMAIN}.crt owner=${USER} group=${USER} mode=0644" -u ${USER} -i hosts
                        """ 
                    }
                }
            }
        }
    }
}