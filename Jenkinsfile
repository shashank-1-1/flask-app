pipeline {
    agent any
    environment {
        PROJECT_NAME = 'flask-app'
        OPENSHIFT_SERVER = 'https://api.cacheocpnode.cacheocp.com:6443'
        OPENSHIFT_TOKEN = credentials('openshift-token')  // Add token in Jenkins credentials
        GIT_REPO = 'https://github.com/shashank-1-1/flask-app.git'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }
        stage('Build Image') {
            steps {
                script {
                    // Login to OpenShift (use 'bat' for Windows)
                    bat "oc login ${OPENSHIFT_SERVER} --token=${OPENSHIFT_TOKEN} --insecure-skip-tls-verify"
                    // Switch to project
                    bat "oc project ${PROJECT_NAME}"
                    // Build and push image to OpenShift registry
                    bat "oc new-build --binary --name=flask-app --strategy=docker"
                    bat "oc start-build flask-app --from-dir=. --follow"
                }
            }
        }
        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Apply DeploymentConfig, Service, and Route
                    bat "oc apply -f openshift/deploymentconfig.yaml"
                    bat "oc apply -f openshift/service.yaml"
                    bat "oc apply -f openshift/route.yaml"
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    bat "oc rollout status dc/flask-app"
                }
            }
        }
    }
}
