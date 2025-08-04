//Con il Jenkinsfile puoi definire il tuo pipeline in modo dichiarativo e farla eseguire da Jenkins esattamente come se stessi definendo in Item di tipo pipeline dove scriviamo la pipeline stessa!

pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '29970ded-725b-4187-839c-dacaedd58c3c'
        //MAI SALVARE SECRETS/JST API TOKEN IN JENKINSFILE, MA DIRETTAMENTE IN JENKINS CREDENTIALS. Accediamo poi alle JenkinsCredentialis tramite la funzione credentials('id-della-credential')
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') //Netlifi si aspetta esattamente questo nome per il token di autenticazione: NETLIFY_AUTH_TOKEN
        REACT_APP_VERSION = '1.2.3' //Versione dell'applicazione React, può essere utile per il deploy
    }

    stages {

        
        stage('Build') {
            
            //Per questo stage Build per eseguire steps devo usare un agente che è ina immagine docker di node
            //Jenkins chiede a docker di runnare l'immagine, eseguire i comandi in steps e rimuovere il container al termine
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
        

        //Esecuzione stages in parallelo
        //Si crea uno stage che raggruppa gli altri due stage e li esegue in parallelo. Per fallo dobbiamo racchiudere in una closure (le parentesi graffe) e usare la parola chiave parallel
        stage('Unit Test && E2E') {
            
            parallel {

                stage('Unit Test') {

                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    //npm test è un comando che esegue gli script di test definiti nel package.json e genera junit.xml nel formato junit nella folder test-results
                    steps {
                        sh '''
                            echo "Running Unit Test..."
                            touch build/index.html
                            npm test
                        '''                    
                    }

                    //Pubblichiamo junit.xml (e lo archiviamo). Questo farà comparire in jenkins il report dei test
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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

                    //Installare il plugin html publisher per pubblicare il report dei test in jenkins
                    //possiamo pubblicarlo 
                    steps {
                        sh '''
                            echo "E2E Test..."
                            npm install serve
                            #node_modules/.bin/serve -s build mette il server in run bloccando la pipeline in quanto il server rimane in esecuzione non esce(come è ovvio che sia). Con & lo mettiamo in background
                            node_modules/.bin/serve -s build &
                            #Attendo che il server parta
                            sleep 10
                            npx playwright test --reporter=html
                        '''                    
                    }

                    //Pubblichiamo l'html report
                    post {
                        always {

                            //Codice generato dentro jenkins in Cinfigurazione Job -> Pipeline Syntax -> Publish HTML reports
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
    

                }

            }

        }

        stage('Depoly Staging') {

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true

                }
            }

            //Installare il plugin html publisher per pubblicare il report dei test in jenkins
            //possiamo pubblicarlo 
            steps {
                sh '''
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify status
                    
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json #--json ritorna il result in json in deploy-output.json grazie all' operatore(linux) di redirezione dell'output
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                    
                    npx playwright test --reporter=html
                '''                    
            }

            //Pubblichiamo l'html report
            post {
                always {

                    //Codice generato dentro jenkins in Cinfigurazione Job -> Pipeline Syntax -> Publish HTML reports
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }


        }

        /*
        stage('Approval') {
            steps {
                script {
                    timeout(time: 30, unit: 'SECONDS') {
                        //Chiediamo l'approvazione manuale per procedere con il deploy in produzione
                        input message: 'Approve deployment to production?', ok: 'Deploy to Prod'
                    }
                }
            }
        }
        */

        stage('Depoly Prod') {

            environment {
                CI_ENVIRONMENT_URL = "https://magnificent-hamster-02cbb4.netlify.app"
            }

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            //Installare il plugin html publisher per pubblicare il report dei test in jenkins
            //possiamo pubblicarlo 
            steps {
                sh '''
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
    
                    #npm install netlify-cli # Non serve installare netlify-cli, è già installato nello stage di Depoly Staging con reuseNode true -> nei node_modules del workspace è gia presente
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test  --reporter=html
                '''                    
            }

            //Pubblichiamo l'html report
            post {
                always {

                    //Codice generato dentro jenkins in Cinfigurazione Job -> Pipeline Syntax -> Publish HTML reports
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }


        }


    }

}