pipeline{
    agent any
    environment{
        ECR_FRONTEND="515210271098.dkr.ecr.eu-west-2.amazonaws.com/sample-mernapp-frontend"
        ECR_BACKEND="515210271098.dkr.ecr.eu-west-2.amazonaws.com/sample-mern-hello-svc"
        ECR_BACKEND_PROFILE_SERVICE="515210271098.dkr.ecr.eu-west-2.amazonaws.com/sample-mernapp-profile-svc"
        CODECOMMIT_BRANCH='main'
        CODECOMMIT_URL='https://git-codecommit.eu-west-2.amazonaws.com/v1/repos/sample-mern-app'
        CODECOMMIT_CREDENTIALS='aws-code-commit-credentials'
        AWS_KEYS='access-keys'
    }
    stages{
        stage('checkout code'){
            steps{
                git branch: env.CODECOMMIT_BRANCH, url: env.CODECOMMIT_URL, credentialsId: env.CODECOMMIT_CREDENTIALS
            }
        }
        stage("Creating .env file for services"){
            steps{
                script{
                    def port_list=[3001,3002]
                    def envContent="""
                    MONGO_URL="mongodb+srv://Ishwari:Ishwari@cluster1.2x5egls.mongodb.net/"
                    PORT=3002
                    """
                    writeFile(file: './backend/helloService/.env', text: envContent.trim())
                    writeFile(file: './backend/profileService/.env', text: envContent.trim())
                }
            }
        }
        
        stage("Build frontend docker image"){
            steps{
                script{
                    docker.build("${env.ECR_FRONTEND}:${env.BUILD_ID}", './frontend/')
                }
            }
            post{
                always{
                    script{
                        docker.withRegistry(${env.ECR_FRONTEND}, ${env.AWS_KEYS}) {
                            
                            docker.image("${env.ECR_FRONTEND}:${env.BUILD_ID}").push()
                        }
                    }
                }
                
            }

        }
        stage("Build helloService docker image"){
            steps{
                script{
                    docker.build("${env.ECR_BACKEND_HELLO_SERVICE}:${env.BUILD_ID}", './backend/helloService/')
                }
            }
            post{
                always{
                    script{
                        docker.withRegistry(${env.ECR_BACKEND_HELLO_SERVICE}, ${env.AWS_KEYS}) {
                            
                            docker.image("${env.ECR_BACKEND_HELLO_SERVICE}:${env.BUILD_ID}").push()
                        }
                    }
                }
                
            }
        }
        stage("Build profileService docker image"){
            steps{
                script{
                    docker.build("${env.ECR_BACKEND_PROFILE_SERVICE}:${env.BUILD_ID}", './backend/profileService/')
                }
            }
            post{
                always{
                    script{
                        docker.withRegistry(${env.ECR_BACKEND_PROFILE_SERVICE}, ${env.AWS_KEYS}) {
                            
                            docker.image("${env.ECR_BACKEND_PROFILE_SERVICE}:${env.BUILD_ID}").push()
                        }
                    }
                }
                
            }
        }
       
        
    }
}