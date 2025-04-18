pipeline{
    agent any
    environment{
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "us-east-1"
        SONAR_SCANNER_HOME = tool 'sonarqube-scanner702';
    }
    stages{
        stage("checkout code"){
            steps{
                git branch: 'main', credentialsId: 'github_jenkins', url: 'https://github.com/drissahdadouch/Web_app.git'
            }
        }
        
        stage("check dependency"){
            parallel{
                
             stage('NPM Dependency Audit') {
                    steps {
                        dir('frontend'){
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                        }
                        dir('backend'){
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                        }
                    }
                }
        
        stage("owasp_check"){
            steps{
               dependencyCheck additionalArguments: '--format ALL', odcInstallation: 'owasp_dependency_check'
            
                }
               
              }
           }
        }
        stage('unit testing'){
            steps{
             dir('frontend'){
                 sh'npm install'
                  sh' npm test' 
              } 
             junit allowEmptyResults: true, stdioRetention: '', testResults: 'dependency-check-junit.xml'
            }
        }    
       
        stage("code quality testing with sonarqube"){
            steps{
                sh'''
                $SONAR_SCANNER_HOME/bin/sonar-scanner \
                   -Dsonar.projectKey=pfa_pipeline \
                   -Dsonar.sources=. \
                   -Dsonar.host.url=http://127.0.0.1:9000 \
                   -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info \
                   -Dsonar.token=sqp_c938e562bcb28ee470127ee0165685b15d6ffd0c
                '''
            }
        } 
        stage("Build Docker images"){
            steps{
                dir('frontend'){
                    sh ''' 
                    docker build -t drissahd/frontend_app .
                    '''
                }
                dir('backend'){
                  sh ''' 
                    docker build -t drissahd/backend_app .
                    '''  
                }
            }
        }
        stage("scanning docker images with trivy"){
            steps{
                sh '''
                trivy image  --severity HIGH,CRITICAL --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o trivy-scan-frontend-report.html drissahd/frontend_app
                
                trivy image  --severity HIGH,CRITICAL --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o trivy-scan-backend-report.html drissahd/backend_app
               '''
            }
        }
        stage("push docker images to Dockerhub"){
            steps{
                withDockerRegistry(credentialsId: 'Dockerhub_token', url: 'https://index.docker.io/v1/'){
               sh' docker push drissahd/frontend_app'
               sh' docker push drissahd/backend_app'
              }
            }
        } 
        stage("connect to ec2 instance"){
            steps{
                sshagent(['204.236.201.79']) {
                sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@204.236.201.79 "
                        docker pull drissahd/frontend_app
                        docker stop frontend_app || true
                        docker rm frontend_app || true
                        docker run -d --name frontend_app -p 3000:3000 drissahd/frontend_app:latest
                    " 
                '''
                }
            }
        } 
    } 
}
