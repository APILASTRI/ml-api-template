node {
    def registry = 'ricoms858/divorce-predictor'
    stage('Checking out git repo') {
      echo 'Checkout...'
      checkout scm
    }
    stage('Linting') {
        echo 'Linting...'
        sh 'docker run --rm -i hadolint/hadolint:v1.17.6-3-g8da4f4e-alpine < Dockerfile'
    }
    stage('Build image') {
        echo 'Building Docker image...'
        sh 'docker build -f Dockerfile -t ${registry} .'
    }
    stage('Push image') {
        withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){
            sh 'docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}'
            sh 'docker tag ${registry} ${registry}'
            sh 'docker push ${registry}'
        }
        sh 'make upload'
    }
    stage('set current kubectl context') {
        dir ('./') {
            withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                sh "aws eks --region us-east-2 update-kubeconfig --name CapstoneEKS"
            }
        }
    }
    stage('Deploy Kubernetes') {
        dir ('./') {
            withAWS(credentials: 'personal-devops', region: 'us-east-1') {
                sh "kubectl apply -f aws/aws-auth-cm.yaml"
                sh "kubectl set image deployments/capstone-app capstone-app=${registry}:latest"
                sh "kubectl apply -f aws/capstone-app-deployment.yml"
                sh "kubectl get nodes"
                sh "kubectl get pods"
                sh "aws cloudformation update-stack --stack-name udacity-capstone-nodes --template-body file://aws/worker_nodes.yml --parameters file://aws/worker_nodes_parameters.json --capabilities CAPABILITY_IAM"
            }
        }
    }
    stage("Cleaning up") {
        echo 'Cleaning up...'
        sh 'docker system prune'
    }
}