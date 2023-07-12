pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "elfiadylclc" // replace this with your docker-id
DOCKER_IMAGE_CAST = "datascientestcast"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {

        stage(' Docker run cast db '){   
             steps {
                script {
                sh '''
                docker rm -f cast_db
                docker run -d --volume postgres_data_cast:/var/lib/postgresql/data/ -e POSTGRES_USER='cast_db_username' -e POSTGRES_PASSWORD='cast_db_password' -e POSTGRES_DB='cast_db_dev' --name cast_db postgres:12.1-alpine
                sleep 10
                '''
                }
              }
        }    

        stage(' Docker Build Cast'){

                    steps {
                        script {
                        sh '''
                        docker rm -f cast_service
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service/
                        sleep 6
                        '''
                        }
                    }
        }

         stage(' Docker run Cast'){
           
                        steps {
                            script {
                            sh '''
                            docker run -d -p 8002:8000 --volume ./cast-service/:/app/ -e DATABASE_URI='postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev' --name cast_service $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG 
                            sleep 10
                            '''
                            }
                        }
        }    

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                '''
                }
            }

        }
        
        stage('Deploiement db en dev'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp castdb/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install castdb castdb --values=values.yml --namespace dev
                                '''
                                }
                            }

         }
      

        stage(' Deploiement app en dev'){

                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp cast/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install cast cast --values=values.yml --namespace dev
                                '''
                                }
                            }

        }                 

        stage(' Deploiement db en qa'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp castdb/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install castdb castdb --values=values.yml --namespace qa
                                '''
                                }
                            }

        }
		

        stage(' Deploiement app en qa'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp cast/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install cast cast --values=values.yml --namespace qa
                                '''
                                }
                            }

        }

        stage(' Deploiement cast db en staging'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp castdb/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install castdb castdb --values=values.yml --namespace staging
                                '''
                                }
                            }
        }        

        stage(' Deploiement en staging'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp cast/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install cast cast --values=values.yml --namespace staging
                                '''
                                }
                            }

                
        }

        stage(' Deploiement cast db en prod'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                timeout(time: 15, unit: "MINUTES") {
                                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                                }
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp castdb/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install castdb castdb --values=values.yml --namespace prod
                                '''
                                }
                            }

        }

        stage(' Deploiement cast en prod'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                timeout(time: 15, unit: "MINUTES") {
                                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                                }
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp cast/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install cast cast --values=values.yml --namespace prod
                                '''
                                }
                            }

        }
}
}
