pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/shrikantdandge665/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                 sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=Ekart \
                 -Dsonar.java.binaries=. '''
              }
            }
        }
        
        stage('OWASP depency check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploy to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy  -DskipTests=true"
                }
            }
        }
        
        stage('Build &tag') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker build -t shrikantdandge7/ekart:latest -f docker/Dockerfile ."
                  }
                }
            }
        }
        
        stage('trivy scan') {
            steps {
                sh "trivy image shrikantdandge7/ekart:latest > trivy-report.txt"
            }
        }
        
        stage('Docker image push') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker push shrikantdandge7/ekart:latest"
                  }
                }
            }
        }
        
        stage('Kubernets deploy') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://172.31.59.35:6443']]) {
                     sh "kubectl apply -f deploymentservice.yml"
                     sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
