pipeline {
    agent any
    stages {
        stage('Dependencies installation') {
            agent {
                docker {
                    image 'node:22'
                    reuseNode true
                    //args '-v /var/jenkins_home/cache:/root/.npm'
                }
            }
            stages {
                stage('Install commands') {
                    steps {
                        sh 'npm install'
                    }
                }
                stage('Testing app') {
                    steps {
                        sh 'npm run test:cov'
                    }
                }
                stage('Building') {
                    steps {
                        sh 'npm run build'
                    }
                }
            }
        }
        stage('Quality Assurance'){
            agent{
                docker {
                    image 'sonarsource/sonar-scanner-cli'
                    args '--network=devops-infra_default'
                    reuseNode true
                }
            }
            stages{
                stage('Uploading code to sonarqube'){
                    steps{
                        withSonarQubeEnv('SonarQube'){
                            sh 'sonar-scanner'
                        }
                    }
                }

                stage('Quality Gate'){
                    steps{
                        timeout(time: 30, unit: 'SECONDS') {
                            script {
                                    def qg = waitForQualityGate()
                                    if (qg.status != 'OK') {
                                        error "La puerta de calidad no paso: ${qg.status}"
                                    }
                            }
                        }
                    }
                }
            }
        }
        stage('Image building and delivery stage'){
            steps{
                script {
                    docker.withRegistry('http://localhost:8082', 'nexus-credentials') {
                        sh 'docker build -t backend-test .'
                        sh 'docker tag backend-test localhost:8082/backend-test:latest'
                        sh 'docker push localhost:8082/backend-test:latest'
                    }

                    docker.withRegistry('http://localhost:8082', 'nexus-credentials') {
                        sh 'docker tag backend-test localhost:8082/backend-test:${env.BUILD_NUMBER}'
                        sh 'docker push localhost:8082/backend-test:${env.BUILD_NUMBER}'
                    }
                }

                    
            }
        }
        //stage('Building'){
        //    steps {
        //        echo 'Another building...'
        //    }
        //}
    }
}