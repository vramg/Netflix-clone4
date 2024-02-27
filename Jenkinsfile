pipeline {
    agent any
    tools {
        jdk 'JDK'
        nodejs 'NODEJS'
    }
    environment {
        SCANNER_HOME = tool 'Sonar'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/chennareddy5/Netflix-clone.git'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix -Dsonar.projectKey=Netflix"
                    }
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'OSWAP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker', toolName: 'Docker') {
                        sh "docker build --build-arg TMDB_V3_API_KEY=40161866f03f47f679d2847d116fb7cf-t -t netflix ."
                        sh "docker tag netflix chennareddy12/netflix:latest"
                        sh "docker push chennareddy12/netflix:latest"
                    }
                }
            }
        }
        stage("Trivy") {
            steps {
                sh "trivy image chennareddy12/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container') {
            steps {
                sh 'docker run -d --name netflix -p 8081:80 chennareddy12/netflix:latest'
            }
        }
    }
}
