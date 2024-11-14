pipeline {
    agent any
    environment {
        PROJECT_NAME = 'flask-app'
        OPENSHIFT_SERVER = 'oc login --token=sha256~0tNkprYjyXVDmtS4NEqJL3eS1ZeGFvr-Ef10_4duKNE --server=https://api.cacheocpnode.cacheocp.com:6443'
        OPENSHIFT_TOKEN = 'sha256~0tNkprYjyXVDmtS4NEqJL3eS1ZeGFvr-Ef10_4duKNE'  // Add token in Jenkins credentials
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
                    // Login to OpenShift
                    sh "oc login ${OPENSHIFT_SERVER} --token=${OPENSHIFT_TOKEN} --insecure-skip-tls-verify"
                    // Switch to project
                    sh "oc project ${PROJECT_NAME}"
                    // Build and push image to OpenShift registry
                    sh "oc new-build --binary --name=flask-app --strategy=docker"
                    sh "oc start-build flask-app --from-dir=. --follow"
                }
            }
        }
        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Apply DeploymentConfig, Service, and Route
                    sh "oc apply -f openshift/deploymentconfig.yaml"
                    sh "oc apply -f openshift/service.yaml"
                    sh "oc apply -f openshift/route.yaml"
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    sh "oc rollout status dc/flask-app"
                }
            }
        }
    }
}
