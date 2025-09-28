pipeline {
    agent any

    environment {
        REGISTRY = "kaleem21"
        IMAGE_PREFIX = "otel-demo"   // prefix for all microservice images
        KUBE_CONFIG = credentials('kubeconfig-credentials-id') // Jenkins credential
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/kbm2108/opentelemetry-demo.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // build all microservices under src/ directory
                    def services = sh(script: "ls src", returnStdout: true).trim().split("\n")
                    for (svc in services) {
                        def imageName = "${REGISTRY}/${IMAGE_PREFIX}-${svc}:${BUILD_NUMBER}"
                        echo "Building image for service: ${svc}"
                        dir("src/${svc}") {
                            docker.build(imageName, ".")
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry("https://index.docker.io/v1/", "dockerhub-credentials-id") {
                        def services = sh(script: "ls src", returnStdout: true).trim().split("\n")
                        for (svc in services) {
                            def imageName = "${REGISTRY}/${IMAGE_PREFIX}-${svc}:${BUILD_NUMBER}"
                            docker.image(imageName).push()
                            docker.image(imageName).push("latest")
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-credentials-id']) {
                    sh """
                        kubectl apply -f kubernetes/otel-demo.yaml
                        kubectl rollout status deployment --all
                    """
                }
            }
        }

        stage('Post-Deployment Checks') {
            steps {
                sh "kubectl get pods -o wide"
                sh "kubectl get svc -o wide"
            }
        }
    }

    post {
        success {
            echo "OpenTelemetry Demo deployed successfully ✅"
        }
        failure {
            echo "Pipeline failed ❌"
        }
    }
}
