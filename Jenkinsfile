pipeline {
    agent any

    stages {
        stage('Deploy To Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'saurabh-eks', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://61C973E03CC3760B18D9F26DB770F7FC.gr7.ap-south-1.eks.amazonaws.com']]) {
                    sh "kubectl apply -f deployment-service.yml"
                    sleep 60
                }
            }
        }
        
        stage('verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'saurabh-eks', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://61C973E03CC3760B18D9F26DB770F7FC.gr7.ap-south-1.eks.amazonaws.com']]) {
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
