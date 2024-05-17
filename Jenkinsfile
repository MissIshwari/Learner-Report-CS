pipeline{
    agent any
    environment{
        DOCKER_FRONTEND="misscoder21/learner-report-frontend"
        DOCKER_BACKEND="misscoder21/learner-report-backend"
        GIT_BRANCH="main"
        GIT_URL="https://github.com/MissIshwari/Learner-Report-CS"
        // AWS_KEYS=credentials('access-keys')
        DOCKER_HUB_KEY = credentials('ishwari-docker-hub')
        //creating a dockerhub credentials in jenkins for github private credentials
    }
    stages{
        stage('checkout code'){
            
            steps{
                git branch: env.GIT_BRANCH, url: env.GIT_URL
            }
        }
        
        stage("Build frontend docker image"){
            steps{
                script{
                    docker.build("${env.DOCKER_FRONTEND}:${env.BUILD_ID}", './learnerReportCS_frontend/')
                }
            }
            post{
                always{
                    script{
                         withCredentials([usernamePassword(credentialsId: 'ishwari-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo -n ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    }
                    withCredentials([usernamePassword(credentialsId: 'ishwari-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        docker.withRegistry('https://index.docker.io/v1/', 'ishwari-docker-hub') {
                            docker.image("${env.DOCKER_FRONTEND}:${env.BUILD_ID}").push()
                            // docker.image("${env.DOCKER_FRONTEND}:${env.BUILD_ID}").push()
                        }
                    }
                    }
                }
                
            }

        }
        stage("Build backend docker image"){
            steps{
                script{
                    docker.build("${env.ECR_BACKEND_HELLO_SERVICE}:${env.BUILD_ID}", './learnerReportCS_backend/')
                }
            }
            post{
                always{
                    script{
                         withCredentials([usernamePassword(credentialsId: 'ishwari-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo -n ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    }
                    withCredentials([usernamePassword(credentialsId: 'ishwari-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        docker.withRegistry('https://index.docker.io/v1/', 'ishwari-docker-hub') {
                            docker.image("${env.DOCKER_BACKEND}:${env.BUILD_ID}").push()
                            // docker.image("${env.DOCKER_IMAGE_RESUME_BUILDER_FRONTEND}:${env.BUILD_ID}").push()
                        }
                    }
                    }
                }
                
            }
        }
        stage("Connect to EKS"){
            steps{
                script{
                    withCredentials([aws(credentialsId: 'ishwari-aws-key', region:'eu-west-2')]) {
                        
                        def eksClusterExists = sh(script: 'aws eks describe-cluster --name ishwari-eks-cluster-1 --region eu-west-2', 
                        returnStatus: true) == 0
                        if(!eksClusterExists){
                            sh '''
                            eksctl create cluster --name ishwari-eks-cluster-1 --region eu-west-2 --nodegroup-name standard-workers --node-type t3.micro --nodes 2 --nodes-min 1 --nodes-max 3
                            '''
                        }
                    }
                }
            }
        }
        stage('Update the EKS cluster') {
            steps {
                script {
                    def clusterStatus = sh(script: "eksctl get cluster --name LearnerReportCSclusterNEW --region ap-south-1", returnStatus: true)
                    if (clusterStatus != 0) {
                        echo "Cluster does not exist, creating one."
                        sh "eksctl create cluster --name LearnerReportCSclusterNEW --region ap-south-1"
                    } else {
                        echo "Cluster exists, moving on with deployment."
                    }
                    sh "aws eks update-kubeconfig --name LearnerReportCSclusterNEW --region ap-south-1"
                    sh "helm upgrade --install learnreportcs LearnerReportCS-helm"
                }
            }
        }
       
        
    }
}