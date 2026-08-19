pipeline {

    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    tools {
        maven 'maven3'
        jdk 'jdk-8'
    }

    stages {

        stage('git checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ajayM1988/Ekart_project.git'
            }
        }

        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('unit tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh """
                        ${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'nvd-api-key',
                        variable: 'NVD_API_KEY'
                    )
                ]) {
                    sh '''
                        mvn org.owasp:dependency-check-maven:8.4.0:check \
                        -DnvdApiKey="$NVD_API_KEY"
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('deploy to Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'global-maven',
                    jdk: 'jdk-8',
                    maven: 'maven3',
                    traceability: true
                ) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('build and Tag docker image') {
            steps {
                sh 'docker build -t ajay676/ekart:latest -f docker/Dockerfile .'
            }
        }

        stage('Push image to Hub') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'dockerhub-pwd',
                        variable: 'dockerhubpwd'
                    )
                ]) {
                    sh '''
                        echo "$dockerhubpwd" | docker login \
                        -u ajay676 \
                        --password-stdin

                        docker push ajay676/ekart:latest
                    '''
                }
            }
        }

        stage('EKS and Kubectl configuration') {
            steps {
                sh 'aws eks update-kubeconfig --region ap-south-1 --name project-cluster'
            }
        }

        stage('Deploy to k8s') {
            steps {
                sh 'kubectl apply -f deploymentservice.yml'
            }
        }
    }
}
