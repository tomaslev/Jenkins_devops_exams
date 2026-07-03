pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "bonwitt"  // replace this with your docker-id
        SERVICES = "cast-service,movie-service"  // comma-separated list
        NAMESPACES = "dev,qa,staging,prod"
        IMAGE_TAG = "${GIT_BRANCH.tokenize('/').last()}-${GIT_COMMIT.take(7)}" 
    }
    agent any // Jenkins will be able to select all available agents
    stages {
        stage('Docker Build'){ // docker build image stage
            steps {
                script {
                    // Loop through each service and build
                    SERVICES.split(',').each { service ->
                        sh """
                        docker build -t ${DOCKER_ID}/${service}:${IMAGE_TAG} ./${service}
                        sleep 6
                        """
                    }
                }
            }
        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }
            steps {
                script {
                    sh "docker login -u $DOCKER_ID -p $DOCKER_PASS"
                    SERVICES.split(',').each { service ->
                        sh "docker push ${DOCKER_ID}/${service}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                        mkdir -p .kube
                        cat $KUBECONFIG > .kube/config
                    """
                    SERVICES.split(',').each { service ->
                        sh """
                            helm upgrade --install ${service} ./charts \
                            --set image.repository=${DOCKER_ID}/${service} \
                            --set image.tag=${IMAGE_TAG} \
                            -n dev
                        """
                    }
                }
            }
        }

        stage('Deploy to QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                        mkdir -p .kube
                        cat $KUBECONFIG > .kube/config
                    """
                    SERVICES.split(',').each { service ->
                        sh """
                            helm upgrade --install ${service} ./charts \
                            --set image.repository=${DOCKER_ID}/${service} \
                            --set image.tag=${IMAGE_TAG} \
                            -n qa
                        """
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                        mkdir -p .kube
                        cat $KUBECONFIG > .kube/config
                    """
                    SERVICES.split(',').each { service ->
                        sh """
                            helm upgrade --install ${service} ./charts \
                            --set image.repository=${DOCKER_ID}/${service} \
                            --set image.tag=${IMAGE_TAG} \
                            -n staging
                        """
                    }
                }
            }
        }

        stage('Approval Gate & Deploy to Prod') {
            when {
                branch 'master'  // Only master can reach this stage
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Deploy to production?', ok: 'Deploy'
                }
                script {
                    sh """
                        mkdir -p .kube
                        cat $KUBECONFIG > .kube/config
                    """
                    SERVICES.split(',').each { service ->
                        sh """
                            helm upgrade --install ${service} ./charts \
                            --set image.repository=${DOCKER_ID}/${service} \
                            --set image.tag=${IMAGE_TAG} \
                            -n prod
                        """
                    }
                }
            }
        }
    }
}

