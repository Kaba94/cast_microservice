pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "ikaba94" // replace this with your docker-id
DOCKER_IMAGE = "exam"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build cast service'){ // docker build image prod
            steps {
                script {
                sh '''
                 docker rm -f dev-cast
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE-cast:$DOCKER_TAG cast-service/
                sleep 6
                '''
                }
            }
        }
        stage(' Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 80:80 --name dev-cast $DOCKER_ID/$DOCKER_IMAGE-cast:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Docker Push cast'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE-cast:$DOCKER_TAG
                '''
                }
            }

        }

stage('Deploiement en dev'){
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
                cp exam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install test-chart exam --values=values.yml --namespace dev
                '''
                }
            }

        }
}
}
