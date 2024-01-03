def imageName="robert89111911/backend"
def dockerTag=""
def dockerRegistry=""
def registryCredential="dockerhub"

pipeline {
    agent {
        label 'agent'
    }
    
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = "1"
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Get Code from repo') {
            steps {
                checkout scm    // git branch: 'main', url: 'https://github.com/Robert89111911/Backend.git'
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'pip3 install -r requirements.txt'
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Build application image') {
            steps {
                script {
                    dockerTag = "RC-${env.BUILD_ID}"
                    applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
        stage('Pushing image to Artifactory') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredential") {
                        applicationImage.push()
                        applicationImage.push("latest")
                    }
                }
            }
        }
    }

    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
        success {
            build job: 'app_of_apps', parameters: [ string(name: 'backendDockerTag', value: "$dockerTag")], wait: false
        }
    }
}
