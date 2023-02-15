pipeline{

    agent any

    environment{
        AWS_ACCESS_KEY_ID= "AKIA4G6D7JOBOAYFHQNW" 
        AWS_SECRET_ACCESS_KEY= "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"
        AWS_DEFAULT_REGION="us-east-1"
        SKIP="N"
        TERRADESTROY="N"
        FIRST_DEPLOY="Y"
        STATE_BUCKET="testbucket123412341212"
        CLUSTER_NAME="testcluster"
    }

    stages{
        stage("Create Terraform State Buckets"){
            when{
                environment name:'FIRST_DEPLOY',value:'Y'
                environment name:'TERRADESTROY',value:'N'
                environment name:'SKIP',value:'N'
            }
            steps{
                   sh'''
                aws s3 mb s3://testbucket123412341212'''
                
            }
        }

        stage("Deploy Networking"){
            when{
                environment name:'FIRST_DEPLOY',value:'Y'
                environment name:'TERRADESTROY',value:'N'
                environment name:'SKIP',value:'N'
            }
            stages{
                stage('Validate infra'){
                            steps{
                                sh '''
                                cd networking
                                terraform init
                                terraform validate'''
                            }
                        }
                        stage('apply n/w modules'){
                             
                            steps{
                                sh '''
                                cd networking
                                terraform plan -out outfile
                                terraform apply outfile'''
                            }
                        }
            }
        }

        stage("Deploy Cluster"){
            when{
                environment name:'FIRST_DEPLOY',value:'Y'
                environment name:'TERRADESTROY',value:'N'
                environment name:'SKIP',value:'N'
            }
            stages{
                stage('Validate infra'){
                            steps{
                                sh '''
                                cd cluster
                                terraform init
                                terraform validate'''
                            }
                        }
                        stage('spin up cluster'){
                             
                            steps{
                                sh '''
                                cd cluster
                                terraform plan -out outfile
                                terraform apply outfile'''
                            }
                        }
            }
        }


        stage("Deploy sample app"){
            when{
                environment name:'FIRST_DEPLOY',value:'Y'
                environment name:'TERRADESTROY',value:'N'
                environment name:'SKIP',value:'N'
            }
            steps{
                sh"""
                cd sample_app
                aws eks update-kubeconfig --name ${env.CLUSTER_NAME} 
                kubectl apply -f ng.yml
                """
                sleep 160
            }
        }

        stage('test kubectl'){
                            steps{
                                script {
                                    sh """
                                    cd cluster
                                    aws eks update-kubeconfig --name ${env.CLUSTER_NAME} 
                                    kubectl get pods
                                    kubectl get nodes
                                    """

                                }
                            }
                        }

        stage('Notify on Slack'){
             when{
                environment name:'FIRST_DEPLOY',value:'Y'
                environment name:'TERRADESTROY',value:'N'
                environment name:'SKIP',value:'N'
            }
            steps{
                slackSend botUser: true, channel: '<channel_name>', message: "EKS Cluster successfully deployed. Cluster Name: $CLUSTER_NAME", tokenCredentialId: '<token_name>'
            }
        }



        stage("Run Destroy"){

            when{
                environment name:'TERRADESTROY',value:'Y'
            }
            stages{

                stage("Destroy eks cluster"){
                    when{
                        environment name:'SKIP',value:'Y'
                    }
                    steps{
                        sh '''
                            cd cluster
                            terraform init
                            terraform destroy -auto-approve
                            '''
                    }
                }

                stage("Destroy n/w infra"){
                    when{
                        environment name:'SKIP',value:'Y'
                    }
                    steps{
                        sh '''
                            cd networking
                            terraform init
                            terraform destroy -auto-approve
                            '''
                    }
                }

                stage("Destroy state bucket"){
                    steps{
                         script {
                            sh(returnStdout: true, script: "aws s3 rb s3://'${env.STATE_BUCKET}' --force").trim()                    
                        }
                    }
                }

                //next steps


            }

        }


        




    }

    post { 
        always { 
            cleanWs()
        }
    }





}
