node('wsl') {
    // แก้ไขตรงนี้: ชี้ไปที่โกดังในเครื่องเรา
    def REGISTRY = 'localhost:5001'
    def APP_NAME = 'my-nginx-web'
    def IMAGE_TAG = "${env.BUILD_NUMBER}"
    def FULL_IMAGE_NAME = "${REGISTRY}/${APP_NAME}:${IMAGE_TAG}"

    stage('Checkout SCM') {
        git branch: 'main', url: 'https://github.com/JJoexx/jenkins-web-app.git'
    }

    stage('Build & Push Image') {
        echo "🐳 Build และส่งเข้าโกดังส่วนตัว..."
        sh "docker build -t ${FULL_IMAGE_NAME} -t ${REGISTRY}/${APP_NAME}:latest ."
        sh "docker push ${FULL_IMAGE_NAME}"
        sh "docker push ${REGISTRY}/${APP_NAME}:latest"
    }

    stage('Deploy to Kubernetes') {
        echo "☸️ สั่ง Deploy โดยให้ K8s มาดึงจากโกดังเรา..."
        sh "kubectl apply -f k8s/deployment.yaml"
        sh "kubectl apply -f k8s/service.yaml"
        sh "kubectl apply -f k8s/ingress.yaml"
        // สั่งให้ K8s ใช้ Image จากโกดัง localhost:5001
        sh "kubectl set image deployment/nginx-deployment nginx-container=${FULL_IMAGE_NAME}"
    }

    stage('Verify Deployment') {
        sh "kubectl rollout status deployment/nginx-deployment --timeout=60s"
        sh "kubectl get pods -l app=my-nginx"
    }
}
