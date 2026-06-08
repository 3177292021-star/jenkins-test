pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/3177292021-star/jenkins-test.git'
        DOCKER_IMAGE = '3177292021-star/my-app'
        CHART_PATH = 'helm/my-app'
        RELEASE_NAME = 'my-app'
        K8S_NAMESPACE = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-credentials', url: env.GIT_REPO, branch: 'master'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    def imageTag = env.BUILD_NUMBER
                    sh "docker build -t ${DOCKER_IMAGE}:${imageTag} ."
                    docker.withRegistry('', 'dockerhub-credentials') {
                        sh "docker push ${DOCKER_IMAGE}:${imageTag}"
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    def imageTag = env.BUILD_NUMBER
                    sh """
                        helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                            --namespace ${K8S_NAMESPACE} \
                            --set image.repository=${DOCKER_IMAGE} \
                            --set image.tag=${imageTag} \
                            --wait --timeout 5m
                    """
                }
            }
        }
    }
}
