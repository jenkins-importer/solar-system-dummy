pipeline {
    agent {
        label 'us-west-1-ubuntu-22'
    }
    tools {
        nodejs 'nodejs-22-6-0'
    }
    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_DB_CREDS = credentials('mongo-db-credentials')
        MONGO_USERNAME = credentials('mongo-db-username')
        MONGO_PASSWORD = credentials('mongo-db-password')
    }
    stages {
        stage('Installing Dependencies') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh 'npm install --no-audit'
            }
        }
        stage('Dependency Scanning') {
            parallel {
                stage('NPM Dependency Audit') {
                    // agent {
                    //     docker {
                    //         image 'node:18-alpine'
                    //         args '-u root:root'
                    //     }
                    // }
                    steps {
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                    }
                }
                // stage('OWASP Dependency Check') {
                //     steps {
                //         dependencyCheck additionalArguments: '''
                //             --scan \'./\' 
                //             --out \'./\'  
                //             --format \'ALL\' 
                //             --disableYarnAudit \
                //             --prettyPrint''', odcInstallation: 'OWASP-DepCheck-10'
                //         dependencyCheckPublisher failedTotalMedium: 1, failedTotalLow: 1, failedTotalHigh: 1, pattern: 'dependency-check-report.xml', stopBuild: true
                //         publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                //     }
                // }
            }
        }
        stage('Unit Testing') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                }
            }
            options {
                retry(2)
            }
            steps {
                sh 'npm test'
                junit allowEmptyResults: true, stdioRetention: '', testResults: 'test-results.xml'
            }
        }
        stage('Code Coverage') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future releases', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh  'docker build -t siddharth67/solar-system:$GIT_COMMIT .'
            }
        }
        stage('Trivy Vulnerability Scanner') {
            steps {
                sh  ''' 
                    trivy image siddharth67/solar-system:$GIT_COMMIT \
                        --severity LOW,MEDIUM,HIGH \
                        --exit-code 0 \
                        --quiet \
                        --format json -o trivy-image-MEDIUM-results.json

                    trivy image siddharth67/solar-system:$GIT_COMMIT \
                        --severity CRITICAL \
                        --exit-code 1 \
                        --quiet \
                        --format json -o trivy-image-CRITICAL-results.json
                '''
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-hub-credentials', url: "") {
                    sh  'docker push siddharth67/solar-system:$GIT_COMMIT'
                }
            }
        }
    }
    post {
        always {
            sh "echo post section"
        }
    }
}
