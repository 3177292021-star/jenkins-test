pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: IfNotPresent
    command: ["/busybox/cat"]
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  volumes:
  - name: docker-config
    secret:
      secretName: dockerhub-secret
      items:
      - key: .dockerconfigjson
        path: config.json
"""
        }
    }

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
                git credentialsId: 'github-credentials', url: env.GIT_REPO, branch: 'main'
            }
        }

        stage('Build & Push Image') {
            steps {
                container('kaniko') {
                    script {
                        def imageTag = env.BUILD_NUMBER
                        sh """
                            /kaniko/executor \
                                --context=\$(pwd) \
                                --dockerfile=Dockerfile \
                                --destination=${DOCKER_IMAGE}:${imageTag} \
                                --cache=false
                        """
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
