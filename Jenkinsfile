pipeline {
    agent any

    environment {
        REPO = 'your-docker-repo/your-app'
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    def app = docker.build("${env.REPO}:${env.BUILD_ID}", '--no-cache .')
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    def app = docker.image("${env.REPO}:${env.BUILD_ID}")
                    app.inside {
                        sh 'python -m unittest discover -s tests'
                    }
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', '22c3bd26-a22f-40f1-967d-2a47bfd42157') {
                        def app = docker.image("${env.REPO}:${env.BUILD_ID}")
                        app.push()
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    withCredentials([file(credentialsId: env.KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f k8s/deployment.yaml'
                        sh 'kubectl apply -f k8s/service.yaml'
                    }
                }
            }
        }
    }
}
