pipeline {
    agent any
    parameters {
      text name: 'DOMAIN', defaultValue: 'dimed.com.br', description: 'Domain to update certs'
    }
    stages {
        stage('Get CA-Bundle and Private-Key by Vault') {
            environment {
                VAULT_ADDR = "http://172.17.0.1:8201"
            }
            agent {
                 docker {
                    image 'jaymeriegel/vault-openssl-jq:v1'
                }
            }
            steps {
                withCredentials([[$class: 'VaultTokenCredentialBinding', addrVariable: 'VAULT_ADDR', credentialsId: 'vault-jenkins-role', tokenVariable: 'VAULT_TOKEN', vaultAddr: '$VAULT_ADDR']]) {
                    script {
                        PRIVATE_KEY = sh (
                            script: 'vault read -field=private_key secrets/creds/certificate_ca/${DOMAIN}',
                            returnStdout: true)
                        CA_BUNDLE = sh (
                            script: 'vault read -field=ca_bundle secrets/creds/certificate_ca/${DOMAIN}',
                            returnStdout: true)
                    }
                }
            }
        }
        stage('Provision') {
            steps {
                sshagent (credentials: ['jenkins']) {
                    script {
                        writeFile file: 'ca', text: CA_BUNDLE
                        writeFile file: 'key', text: PRIVATE_KEY
                        sh """
                        #!/bin/bash
                        ansible ${DOMAIN} -m copy -a "src=ca dest=home/certs/teste.txt owner=nodo group=nodo mode=0644" -u nodo -i hosts
                        """ 
                    }
                }
            }
        }
    }
}
