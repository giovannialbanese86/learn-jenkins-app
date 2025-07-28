//Con il Jenkinsfile puoi definire il tuo pipeline in modo dichiarativo e farla eseguire da Jenkins esattamente come se stessi definendo in Item di tipo pipeline dove scriviamo la pipeline stessa!

pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '29970ded-725b-4187-839c-dacaedd58c3c'
        //MAI SALVARE SECRETS/JST API TOKEN IN JENKINSFILE, MA DIRETTAMENTE IN JENKINS CREDENTIALS. Accediamo poi alle JenkinsCredentialis tramite la funzione credentials('id-della-credential')
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') //Netlifi si aspetta esattamente questo nome per il token di autenticazione: NETLIFY_AUTH_TOKEN
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
        

        //Esecuzione stages in sequenza
        /*
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

            //Installare il plugin html publisher per pubblicare il report dei test in jenkins
            //possiamo pubblicarlo 
            steps {
                sh '''
                    npm install serve
                    #node_modules/.bin/serve -s build mette il server in run bloccando la pipeline in quanto il server rimane in esecuzione non esce(come è ovvio che sia). Con & lo mettiamo in background
                    node_modules/.bin/serve -s build &
                    #Attendo che il server parta
                    sleep 10
                    npx playwright test --reporter=html
                '''                    
            }

        }
        */

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
            
            //Per questo stage Depoly per eseguire steps devo usare un agente che è ina immagine docker di node
            //Jenkins chiede a docker di runnare l'immagine, eseguire i comandi in steps e rimuovere il container al termine
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    #echo "WHOAMI"
                    #whoami
                    npm install netlify-cli@20.1.1
                    #node_modules/.bin/netlify --version
                    echo "Deploying to Staging Netlify. Site ID: ${NETLIFY_SITE_ID}"
                    #NETLIFY_AUTH_TOKEN
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir build

                '''                    
            }

        }
        
        stage('Depoly Prod') {
            
            //Per questo stage Depoly per eseguire steps devo usare un agente che è ina immagine docker di node
            //Jenkins chiede a docker di runnare l'immagine, eseguire i comandi in steps e rimuovere il container al termine
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    #echo "WHOAMI"
                    #whoami
                    npm install netlify-cli@20.1.1
                    #node_modules/.bin/netlify --version
                    echo "Deploying to Netlify. Site ID: ${NETLIFY_SITE_ID}"
                    #NETLIFY_AUTH_TOKEN
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir build --prod 

                '''                    
            }

        }

        stage('Prod E2E') {

            environment {
                CI_ENVIRONMENT_URL = "https://magnificent-hamster-02cbb4.netlify.app"
            }

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
                    echo "Prod E2E Test..."
                    npx playwright test --reporter=html
                '''                    
            }

            //Pubblichiamo l'html report
            post {
                always {

                    //Codice generato dentro jenkins in Cinfigurazione Job -> Pipeline Syntax -> Publish HTML reports
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }


        }


    }

}