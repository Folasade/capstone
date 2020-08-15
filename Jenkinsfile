pipeline {
     agent any

     stage('Lint Dockerfile') {
            steps {
                script {
                    docker.image('hadolint/hadolint:latest-debian').inside() {
                            sh 'hadolint ./Dockerfile | tee -a hadolint_lint.txt'
                            sh '''
                                lintErrors=$(stat --printf="%s"  hadolint_lint.txt)
                                if [ "$lintErrors" -gt "0" ]; then
                                    echo "Errors have been found, please see below"
                                    cat hadolint_lint.txt
                                    exit 1
                                else
                                    echo "There are no erros found on Dockerfile!!"
                                fi
                            '''
                    }
                }
            }
        }

     stage('Build Docker Image') {
              steps {
                  sh 'docker build -t capstoneapp-fola .'
              }
         }
         stage('Push Docker Image') {
              steps {
                  withDockerRegistry([url: "", credentialsId: "dockerhub"]) {
                      sh "docker tag capstoneapp-fola folasade/capstoneapp-fola"
                      sh 'docker push folasade/capstoneapp-fola'
                  }
              }
         }
         
     stage('Deploying') {
              steps{
                  echo 'Deploying to AWS...'
                  withAWS(credentials: 'aws-static', region: 'us-west-2') {
                      sh "aws eks --region us-west-2 update-kubeconfig --name eksctl-capstonecluster-cluster"
                      sh "kubectl config use-context arn:aws:eks:us-west-2:900165913645:cluster/eksctl-capstonecluster-cluster"
                      sh "kubectl apply -f capstone-deploy.yml"
                      sh "kubectl get nodes"
                      sh "kubectl get deployments"
                      sh "kubectl get pod -o wide"
                      sh "kubectl get service/capstoneapp-fola"
                  }
              }
        }
        
        
     stage('Checking rollout') {
              steps{
                  echo 'Checking rollout...'
                  withAWS(credentials: 'aws-static', region: 'us-west-2') {
                     sh "kubectl rollout status deployments/capstoneapp-fola"
                  }
              }
        }
        
     stage("Cleaning up") {
              steps{
                    echo 'Cleaning up...'
                    sh "docker system prune"
              }
        }
     }
} 
