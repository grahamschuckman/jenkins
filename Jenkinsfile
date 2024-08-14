pipeline {
    agent any

    environment {
        // Define cluster names, AWS region, and Helm chart
        // You can add as many clusters as needed to CLUSTER_NAMES to onboard them all in one go
        CLUSTER_NAMES = "graham-cluster-demo,graham-cluster-demo2"
        AWS_REGION = 'us-east-1'
        HELM_CHART = 'wiz-sec/wiz-kubernetes-integration'
        PATH = "/Users/graham.schuckman/google-cloud-sdk/bin:/Library/Frameworks/Python.framework/Versions/3.12/bin:/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin:/opt/podman/bin"
    }

    stages {
        // Stage to configure AWS credentials
        stage('Configure AWS Credentials') {
            steps {
                script {
                    // Find out more about the jenkins user and PATH
                    sh "whoami"
                    sh "echo $PATH"
                    // Set AWS credentials and region
                    sh "aws configure set aws_access_key_id <id>"
                    sh "aws configure set aws_secret_access_key <secret>"
                    sh "aws configure set default.region $AWS_REGION"
                    sh "aws configure set default.output json"
                }
            }
        }

        // Stage to set up Helm repository
        stage('Setup Helm Repo') {
            steps {
                script {
                    // Add Helm repository
                    sh 'helm repo add wiz-sec https://charts.wiz.io/'
                    // Update Helm repositories
                    sh 'helm repo update'
                }
            }
        }

        // Stage to deploy to EKS clusters
        stage('Deploy to EKS Clusters') {
            steps {
                script {
                    // Split cluster names
                    def clusterNames = CLUSTER_NAMES.split(',')
                    // Loop through each dictionary in the list
                    clusterNames.each { clusterName ->
                        // Update kubeconfig for the current cluster
                        sh "aws eks update-kubeconfig --region ${AWS_REGION} --name $clusterName"
                        // Get API server endpoint
                        def apiServerEndpoint = sh(script: "aws eks describe-cluster --name $clusterName | jq -r .cluster.endpoint", returnStdout: true).trim()
                        // Display deployment message
                        echo "Deploying Helm chart to EKS cluster $clusterName..."
                        // Upgrade or install Helm chart with specific values
                        sh "helm upgrade --install $clusterName -n wiz --create-namespace --debug -f /Users/graham.schuckman/eks-personal-connector/eksctl-cluster/values.yaml ${HELM_CHART} --set wiz-kubernetes-connector.autoCreateConnector.connectorName=$clusterName --set wiz-kubernetes-connector.autoCreateConnector.apiServerEndpoint=$apiServerEndpoint"
                    }
                }
            }
        }
    }
}