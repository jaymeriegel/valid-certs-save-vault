pipeline {
    agent {
        docker {
            image 'jaymeriegel/vault-openssl-jq:v1'
        }
    }
    parameters {
      choice name: 'DOMAIN', choices: ['dimed.com.br'], description: 'Domínio do certificado.'
      text name: 'CA_BUNDLE', defaultValue: '', description: 'SSL Certificate in x509 format'
      text name: 'KEY', defaultValue: '', description: 'KEY in RSA format'
    }
    stages {
        stage('Valid CA with a Private-Key') {
            steps {
		cat Dockerfile
                script {
                    writeFile file: 'ca', text: params.CA_BUNDLE
                    writeFile file: 'key', text: params.KEY
                                    
                    def isX509File = sh(script: "openssl x509 -in ca", returnStdout: true).trim()
                    def isRsaFile = sh(script: "openssl rsa -in key", returnStdout: true).trim()
                    
                    assert isX509File.contains("BEGIN")
                    assert isRsaFile.contains("BEGIN")

                    def hashOfCa = sh(script: "openssl x509 -noout -modulus -in ca | openssl md5", returnStdout: true).trim()
                    def hashOfKey = sh(script: "openssl rsa -noout -modulus -in key | openssl md5", returnStdout: true).trim()

                    assert hashOfKey.equals(hashOfCa)
                }
            }
        }
        stage('Save CA-Bundle and Key on Vault') {
            steps {
                sh """#!/bin/bash
                            jq -n --arg a "\$(<ca)" '{ca_bundle: \$a}' > ca.json
                            jq -n --arg b "\$(<key)" '{private_key: \$b}' > key.json """
                
                withCredentials([[$class: 'VaultTokenCredentialBinding', addrVariable: 'VAULT_ADDR', credentialsId: 'vault-jenkins-role', tokenVariable: 'VAULT_TOKEN', vaultAddr: 'http://172.17.0.1:8201']]) {
                    script {
                        sh """
                        #!/bin/bash
                        vault kv put secrets/creds/certificate_ca/${DOMAIN} @ca.json @key.json
                             """
                    }
                }
            }
        }
    }
}
