//Con il Jenkinsfile puoi definire il tuo pipeline in modo dichiarativo e farla eseguire da Jenkins esattamente come se stessi definendo in Item di tipo pipeline dove scriviamo la pipeline stessa!

pipeline {
    agent any

    stages {
        stage('Build') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''                    
            }

        }

        stage('Test') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            //npm test è un comando che esegue gli script di test definiti nel package.json e genera junit.xml nel formato junit nella folder test-results
            steps {
                sh '''
                    touch build/index.html
                    npm test
                '''                    
            }

        }

    }
}