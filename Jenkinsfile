pipeline {

    agent none

    environment {

        NODE_ENV="prod1"
        AWS_ACCESS_KEY=""
        AWS_SECRET_ACCESS_KEY=""
        AWS_SDK_LOAD_CONFIG="0"
        BUCKET_NAME="app-digital"
        REGION="us-east-1" 
        PERMISSION=""
        ACCEPTED_FILE_FORMATS_ARRAY=""
        VERSION="1.0.0"
    }


    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    triggers {
        cron('@daily')
    }

    stages{
        stage("Build, Test and Push Docker Image") {
            agent {  
                node {
                    label 'master'
                }
            }
            stages {

                stage('Clone repository') {
                    steps {
                        script {
                            if(env.GIT_BRANCH=='origin/prod'){
                                checkout scm
                            }
                            sh('printenv | sort')
                            echo "My branch is: ${env.GIT_BRANCH}"
                        }
                    }
                }
                stage('Build image'){       
                    steps {
                        script {
                            print "Environment will be : ${env.NODE_ENV}"
                            docker.build("digitalhouse-devops:latest")
                        }
                    }
                }

                stage('Test image') {
                    steps {
                        script {

                            docker.image("digitalhouse-devops:latest").withRun('-p 8030:3000') { c ->
                                sh 'docker ps'
                                sh 'sleep 10'
                                sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'
                                
                            }
                    
                        }
                    }
                }

                stage('Docker push') {
                    steps {
                        echo 'Push latest para AWS ECR'
                        script {
                            docker.withRegistry('http://690998955571.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:awskey') {
                                docker.image('digitalhouse-devops').push()
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Producao') {
            agent {  
                node {
                    label 'prod1'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/prod'){
 
                        environment {

                            NODE_ENV="production"
                            AWS_ACCESS_KEY="123456"
                            AWS_SECRET_ACCESS_KEY="asdfghjkkll"
                            AWS_SDK_LOAD_CONFIG="0"
                            BUCKET_NAME="app-digital"
                            REGION="us-east-1" 
                            PERMISSION=""
                            ACCEPTED_FILE_FORMATS_ARRAY=""
                        }


                        docker.withRegistry('https://690998955571.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:awskey') {
                            docker.image('digitalhouse-devops').pull()
                        }

                        echo 'Deploy para Producao'
                        sh "hostname"
                        sh "docker stop app1"
                        sh "docker rm app1"
                        //sh "docker run -d --name app1 -p 8030:3000 933273154934.dkr.ecr.us-east-1.amazonaws.com/digitalhouse-devops:latest"
                        withCredentials([[$class:'AmazonWebServicesCredentialsBinding' 
                            , credentialsId: 'prods3']]) {
                          sh "docker run -d --name app1 -p 8030:3000 -e NODE_ENV=producao -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e BUCKET_NAME=dh-pi-grupo-lovelace-prod 690998955571.dkr.ecr.us-east-1.amazonaws.com/digitalhouse-devops:latest"
                        }
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://ec2-18-208-209-81.compute-1.amazonaws.com:8030/api/v1/healthcheck'

                    }
                }
            }

        }

    }
}
