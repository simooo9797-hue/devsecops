pipeline {
    agent none
    stages {
        stage('Test') {
            agent {
                kubernetes {
                    yaml '''
                    apiVersion: v1
                    kind: Pod
                    spec:
                      containers:
                      - name: python
                        image: python:3.12-slim
                        command:
                        - sleep
                        args:
                        - infinity
                    '''
                }
            }
            steps {
                container('python') {
                    sh '''
                        pip install -r requirements.txt
                        pytest
                    '''
                }
            }
        }
        stage('Build and Push Image') {
            agent {
                kubernetes {
                    yaml '''
                    apiVersion: v1
                    kind: Pod
                    spec:
                      containers:
                      - name: buildctl
                        image: moby/buildkit:latest
                        command:
                        - sleep
                        args:
                        - infinity
                    '''
                }
            }
            steps {
                container('buildctl') {
                    sh '''
                        buildctl --addr tcp://buildkitd.jenkins:1234 build \
                          --frontend dockerfile.v0 \
                          --opt context=https://github.com/simooo9797-hue/devsecops.git \
                          --output type=image,name=registry.jenkins:5000/devsecops-demo-app:v1,push=true,registry.insecure=true
                    '''
                }
            }
        }
        stage('Deploy') {
            agent {
                kubernetes {
                    yaml '''
                    apiVersion: v1
                    kind: Pod
                    spec:
                      containers:
                      - name: kubectl
                        image: bitnami/kubectl:latest
                        command:
                        - sleep
                        args:
                        - infinity
                        securityContext:
                          runAsUser: 0
                    '''
                }
            }
            steps {
                container('kubectl') {
                    sh '''
                        kubectl rollout restart deployment/devsecops-demo-app -n jenkins
                        kubectl rollout status deployment/devsecops-demo-app -n jenkins
                    '''
                }
            }
        }
    }
}