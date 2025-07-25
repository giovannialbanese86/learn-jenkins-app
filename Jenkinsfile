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
                    #echo "Running tests..."
                    touch build/index.html
                    npm test
                '''                    
            }

        }

        stage('E2E') {

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    //args '-u root:root' //Eseguiamo il container come utente root, Necessario per eseguire i test con Playwright
                }
            }

            
            steps {
                sh '''
                    npm install serve
                    #node_modules/.bin/serve -s build mette il server in run bloccando la pipeline in quanto il server rimane in esecuzione non esce(come è ovvio che sia). Con & lo mettiamo in background
                    node_modules/.bin/serve -s build &
                    #Attendo che il server parta
                    sleep 10
                    npx playwright test playwright 
                '''                    
            }

        }

    }

    //Pubblichiamo junit.xml (e lo archiviamo). Questo farà comparire in jenkins il report dei test
    post {
        always {
            junit 'jest-results/junit.xml'
            //archiveArtifacts artifacts: 'build/**/*', fingerprint: true
        }
    }

}