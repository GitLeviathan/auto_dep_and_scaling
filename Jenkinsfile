pipeline {
    agent any

    environment {
        REPO = 'gitleviathan/auto_dep_and_scaling'
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
                    docker.withRegistry('https://index.docker.io/v1/', '1fb1b907-e449-47db-8925-74be480de4ae') {
                        def app = docker.image("${env.REPO}:${env.BUILD_ID}")
                        app.push()
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f k8s/deployment.yaml --kubeconfig=$KUBECONFIG'
                        sh 'kubectl apply -f k8s/service.yaml --kubeconfig=$KUBECONFIG'
                    }
                }
            }
        }
    }
}
