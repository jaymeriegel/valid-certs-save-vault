pipeline {
    agent any
    parameters {
      text name: 'DOMAIN', defaultValue: 'dimed.com.br', description: 'Domain to update certs'
      text name: 'SPECIFIC_HOST', defaultValue: '', description: 'Type the specific IP of a host to update cert'
    }
    environment {
                CREDENTIALS_ID = "vault-jenkins-role"
                VAULT_ADDR = "http://172.17.0.1:8201"
                CREDENTIALS_SSH = "jenkins"
                VAULT_PATH_TO_GET_SECRETS = "secrets/creds/certificate_ca"
                CERT_DEST_PATH = "/home/vagrant/certs";
                USER = "vagrant"
    }
    stages {
        stage('Get CA-Bundle and Private-Key by Vault') {
            agent {
                 docker {
                    image 'jaymeriegel/vault-openssl-jq:v1'
                }
            }
            steps {
                withCredentials([[$class: 'VaultTokenCredentialBinding', addrVariable: 'VAULT_ADDR', credentialsId: CREDENTIALS_ID, tokenVariable: 'VAULT_TOKEN', vaultAddr: VAULT_ADDR]]) {
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
        stage('Update CA-BUNDLE and Private-Key on Hosts') {
            steps {
                sshagent (credentials: [CREDENTIALS_SSH]) {
                    script {
                        writeFile file: 'ca', text: CA_BUNDLE
                        writeFile file: 'key', text: PRIVATE_KEY

                        if (params.SPECIFIC_HOST == '') {
                            sh """
                            #!/bin/bash
                            ansible ${DOMAIN} -m copy -a "src=ca dest=${CERT_DEST_PATH}/${DOMAIN}.crt owner=${USER} group=${USER} mode=0644" -u ${USER} -i hosts
                            ansible ${DOMAIN} -m copy -a "src=key dest=${CERT_DEST_PATH}/CA-BUNDLE.key owner=${USER} group=${USER} mode=0644" -u ${USER} -i hosts
                            """ 
                        } else {
                            sh """
                            #!/bin/bash
                            ansible ${DOMAIN} -m copy -a "src=ca dest=${CERT_DEST_PATH}/${DOMAIN}.crt owner=${USER} group=${USER} mode=0644" --limit ${SPECIFIC_HOST} -u ${USER} -i hosts
                            ansible ${DOMAIN} -m copy -a "src=key dest=${CERT_DEST_PATH}/CA-BUNDLE.key owner=${USER} group=${USER} mode=0644" --limit ${SPECIFIC_HOST} -u ${USER} -i hosts
                            """ 
                        }
                    }
                }
            }
        }
    }
}
