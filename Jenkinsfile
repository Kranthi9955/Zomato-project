pipeline {
    agent any

    tools {
        jdk 'java17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar'
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kranthi9955/Zomato-project.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
         stage ("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage ("Install dependencies") {
            steps {
                sh 'npm install'
            }
        }
       stage ("OWASP FS SCAN") {
    steps {
        echo "Skipping OWASP due to NVD restriction"
    }
}
        stage ("TRIVY FS SCAN") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
         stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-pass') {
                        sh 'docker build -t image1 .'
                        sh "docker tag image1 sunny5577/loki:sunny557-z1"
                        sh "docker push sunny5577/loki:sunny557-z1"
                    }
                }
            }
        }
        stage ("TRIVY") {
            steps {
                sh 'trivy image sunny5577/loki:sunny557-z1'
            }
        }
        stage ("Deploy to container") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 sunny5577/loki:sunny557-z1'
            }
        }
    }
}
