pipeline {
    agent { label 'slave1' }
    
    tools {
        jdk 'JAVA'
        nodejs 'nodejs'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'bhoomisharma333/zomato-clone'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/bhoomiiisharmaa/Zomato_Clone_Food_Delivery_Web_Application'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=zomato-clone \
                        -Dsonar.projectName="Zomato Clone" \
                        -Dsonar.sources=.
                    '''
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', 
                                odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivy-fs-report.txt'
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
                sh 'docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest'
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image $DOCKER_IMAGE:latest > trivy-image-report.txt'
            }
        }
        
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:$BUILD_NUMBER'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }
        
        stage('Deploy to Container') {
            steps {
                sh 'docker rm -f zomato || true'
                sh 'docker run -d --name zomato -p 3000:3000 $DOCKER_IMAGE:latest'
            }
        }
    }
    
    post {
        always {
            sh 'docker logout || true'
        }
    }
}
