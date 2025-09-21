pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'mysonar'
    }

    stages {
        stage("Clean workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout code") {
            steps {
                git 'https://github.com/Harishreddyoo7/Zomato-Project.git'
            }
        }

        stage("Code Quality Analysis") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=zomato \
                        -Dsonar.projectKey=zomato
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'mysonar'
                }
            }
        }

        stage("Install dependencies") {
            steps {
                sh 'npm install'
            }
        }

        stage("OWASP FS Scan") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Trivy FS Scan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage("Docker Build and Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-password') {
                        sh 'docker build -t image1 .'
                        sh 'docker tag image1 harishoo725/containerzation:Zomato'
                        sh 'docker push harishoo725/containerzation:Zomato'
                    }
                }
            }
        }

        stage("Trivy Image Scan") {
            steps {
                sh 'trivy image harishoo725/containerzation:Zomato'
            }
        }

        stage("Deploy to Container") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 harishoo725/containerzation:Zomato'
            }
        }
    }
}
