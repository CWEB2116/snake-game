pipeline {
    agent any
    stages {
        stage('CLONE GIT REPOSITORY') {
            agent {
                label 'ubuntu-us-appserver-2140'
            }
            steps {
                checkout scm
            }
        }  
 
        stage('SCA-SAST-SNYK-TEST') {
            agent any
            steps {
                script {
                    snykSecurity(
                        snykInstallation: 'Snyk',
                        snykTokenId: 'Snyk_Token',
                        severity: 'critical'
                    )
                }
            }
        }
 
        stage('SonarQube Analysis') {
            agent any
            steps {
                script {
                    def scannerHome = tool 'Scanner'
                    withSonarQubeEnv('Sonar') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=projectgame \
                            -Dsonar.sources=."
                    }
                }
            }
        }
 
        stage('BUILD-AND-TAG') {
            agent any
            steps {
                script {
                    def app = docker.build("ameliamae/sgameimg")
                    app.tag("latest")
                }
            }
        }
 
        stage('POST-TO-DOCKERHUB') {    
            agent any
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_credentials') {
                        def app = docker.image("ameliamae/sgameimg")
                        app.push("latest")
                    }
                }
            }
        }
 
        stage('DEPLOYMENT') {    
            agent any
            steps {
                sh "docker-compose down"
                sh "docker-compose up -d"   
            }
        }
    }
}
